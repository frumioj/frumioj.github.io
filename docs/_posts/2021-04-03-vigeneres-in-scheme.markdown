---
layout: post
title: "Implementing 'le chiffre indéchiffrable' in Scheme"
date: 2021-04-03
categories: scheme racket cryptography crypto
---
I have started to follow Jonathan Katz's excellent [Cryptography](https://www.coursera.org/learn/cryptography) course on Coursera. Of course, as I often do, I have been sidetracked by trying to implement some of the examples mentioned in the course. Which means that I haven't managed to escape the first set of lectures.

The [shift cipher](https://github.com/frumioj/intro_to_crypto/blob/main/shift-cipher.rkt) was easy enough to implement quickly in Racket, and presented few interesting difficulties. Convert an input string into a list, and then _map_ the shift (addition modulo the length of the alphabet) onto each character in the list.

The [Vigenère cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher), however, was more interesting. 

The first interesting thing I learnt about Vigenère is that it wasn't broken for 300 years, resulting in it being given the name "le chiffre indéchiffrable" (the indecipherable cipher). 

Implementing in Racket Scheme also resulted in some challenges, partly arising from the general idea in Scheme that "some assembly may be required" so some basic things that one might expect to exist in a common library in some other language, must be implemented yourself in Scheme. Of course, the genius of Scheme is that "some assembly is required" which helped me learn more than I would have by using another programming language.

My first challenge was "crypto randint" - the idea of a cryptographically secure random number within a range. For Vigenère, all operations are on the alphabet a-z, and your key is chosen from these letters too. So I needed a key of fourteen (the traditional Vigenère key length) alphabetic (ASCII) characters. Racket presents you with a single API call for obtaining one or more cryptographically-secure bytes. A random byte may or may not be an ASCII a-z character. In practice, I couldn't see a better solution that simply continuing to ask for a crypto-secure random byte and checking whether it was in the range, repeatedly, until I had what I needed.

I imagine though, that if the range is very small, this might conceivably take a (relatively) long time.

```scheme
;;; return integers between min and max, derived from a crypto random byte

(define (crypto_randint min max)
  (let ([y (integer-bytes->integer (crypto-random-bytes 1) #f)])
    (cond [(<= min y max) y]
          [else (crypto_randint min max)])))
```
As you can see, the 'cond' forces repeated calls to the same function until an integer within the range is obtained.

This made my Vigenère key generation look like this:

```scheme
;;; generate a Vigenere key of length keysize,
;;; passing in the result so that it can be compared
;;; with the requested keysize as a bound
;;; new bytes are added until the result length matches the keysize

(define (vig-genkey-bytes keysize result)
  (if (= (bytes-length result) keysize)
      result
      (vig-genkey-bytes
       keysize
       (bytes-append result
                     (integer->integer-bytes (crypto_randint 97 122) 1 #f)))))

(define (vig-genkey keysize)
  (vig-genkey-bytes keysize ""))
```

Note the seemingly usual Lispy conceit of passing the result as a parameter in ```vig-genkey-bytes``` and then having a function that passes in the empty string as the initial value of the result, to start the process.

The more complicated work was still to come. 

Because I have spent the better part of 25 years coding in non-functional languages, returning to functional programming has been mind-blowing. My next challenge was to figure out how I would encipher an arbitrary length ciphertext with a fourteen-character key. In Vigenère, the key is "expanded" by repeating the key until its size matches the plaintext message, and then a character from the key is used to shift the corresponding character from the plaintext.

So the solution in a Lisp should have perhaps been totally obvious to me, but eluded me for more than a week as I tried to figure out how I could do a "nested loop" approach in Racket, mapping a function onto each character of the plaintext, yes, but then "inside" that, counting the position in the "key" list. Perhaps one can do that, but I did not succeed in figuring it out, and I totally ignored the obvious solution as I was seduced by memories of procedural programming.

But _eventually_, I came to understand that the first task was to expand my fourteen-character key to the length of the ciphertext. In addition to implementing 'expand', I also did 'truncate'. They look like this:

```scheme
(define (truncate len lst)
  (cond [(null? lst) lst]
        [(> len (length lst)) lst]
        [(= len 0) '()]
        [else
         (cons (car lst) (truncate (sub1 len) (cdr lst)))]))

(define (expand-iter len lst olst nlst)
  (cond [(= len 0) nlst]
        [(null? lst) (expand-iter len olst olst nlst)]
        [else
         (expand-iter (sub1 len) (rest lst) olst (append nlst (list (first lst))))]))

(define (expand len lst)
  (expand-iter len lst lst '()))
```

Implementing ```truncate``` was easy enough. The key notion in ```expand``` is to pass your work in as a parameter and use recursion to do the work of list traversal, while counting down from the required expanded list size. In ```expand-iter``` there are two interesting recursive calls - one when your key list is null, but you haven't yet reached the expanded list size (```(= len 0) nlst```) and the other when you have neither reached the required expanded list size, nor is your key list empty, in which case, you keep appending the current head of the key list to the end of the expanded key list. 

Once I had implemented ```expand``` I could finally implement the encipherment function by using map with _two_ lists - the first list being the plaintext, and the second being the expanded key list. 

```scheme
(define (vig-encr privkey message)
  
  ;;; create the lists for map to work on by converting the input string to a list
  ;;; and in the case of the private key, expand the key so the list is the same
  ;;; size as the message, to make map over two lists possible
  
  (let* ([y (bytes->list message)]
         [k (expand (length y) (bytes->list privkey))])

    ;;; convert from list to string at the end -- might be
    ;;; inefficient?  so can run a map on the list items
        
    (list->bytes
     (map (lambda (chr keychr)
            ;;; use char->integer to get the ASCII char number,
            ;;; and then use the *position* of that letter in the alphabet
            ;;; convert back to a char
            (+
             (char->integer #\a)
             ;;; modulo the keyspace size!
             (modulo
              (+ (-
                  keychr
                  (char->integer #\a))
                 (-
                  chr
                  (char->integer #\a)))
              shift-modulus)))
          y
          k))))
```

The only part of this that's significantly different from the shift cipher implementation is the map call being given two lists, and the creation of the expanded key in:

```scheme
(let* ([y (bytes->list message)]
         [k (expand (length y) (bytes->list privkey))])
```

Note: ```let*``` (as opposed to ```let```) lets (haha) me use a previously bound variable within the binding stage of the ```let``` to create the expanded key list. 

After the weeks-long (kind of pleasant) pain of implementing encryption, decryption came quickly!

```scheme
(define (vig-decr privkey message)
  (let* ([y (bytes->list message)]
         [k (expand (length y) (bytes->list privkey))])
    
    (list->bytes
     (map (lambda (chr keychr)

             (+
              (char->integer #\a)
              (modulo
               (-
                (-
                 chr
                 (char->integer #\a))
                (-
                 keychr
                 (char->integer #\a))
                )
               shift-modulus)))
          y
          k))))
```

**Finishing up...

OK, so now you've seen how to implement the Vigenère cipher in Racket, your assignment is this:

Given the Vigenère key: ```#"glxceeijhxybag"``` and the ciphertext ```#"czagkscolfaiatmvbcm"```, what is the English plaintext?

Hint: you may need to employ a social engineering technique to discover the answer.
