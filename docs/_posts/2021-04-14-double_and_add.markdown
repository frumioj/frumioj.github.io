In this [nice post about elliptic curves](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/), I found the 'double and add' method of doing scalar multiplication, including some Python code. 

Porting these ten lines of code from Python to Scheme should be easy, right?

```python
def bits(n):
    """
    Generates the binary digits of n, starting
    from the least significant bit.

    bits(151) -> 1, 1, 1, 0, 1, 0, 0, 1
    """
    while n:
        yield n & 1
        n >>= 1

def double_and_add(n, x):
    """
    Returns the result of n * x, computed using
    the double and add algorithm.
    """
    result = 0
    addend = x

    for bit in bits(n):
        if bit == 1:
            result += addend
        addend *= 2

    return result
```

Ah. Well, notice the use of a "generator". I immediately wondered whether Racket had generators, and once I'd looked that up, I got this, for the Python 'bits' method.

```scheme
(define bits
  (generator (n)
             ;;; use the assignment to create the start of the "loop"
             ;;; where this comes back to after a yield
             (let loop ([this n])
               (begin
                 (when (not (zero? this))
                   ;;; give up a value at this point
                   (yield (bitwise-and this 1))
                   (loop (arithmetic-shift this -1)))))))
```

Once I understood the syntax, I got this working pretty quickly. It just didn't do what I needed. Instead of making multiple calls to the generator, and perhaps looking at more of the "spirit" of the Python code, rather than its actual implementation, I wondered why not just create a _list_ of bits?

```scheme
;;; Take a number and generate a list of 1s and 0s
;;; corresponding to whether a bit is set in the
;;; binary representation of n
;;; using bit shifting

(define (bitz-iter n lizt)
  (cond [(zero? n) lizt]
        [else
         (bitz-iter (arithmetic-shift n -1) (cons (bitwise-and n 1) lizt))]))

(define (bitz n)
  (reverse (bitz-iter n '())))
```

How does this compare to the Python generator approach? The Scheme version above seems to be O(2k) where k is the number of bit-shift operations on n, or the length of the list produced. Reversing the list is O(k) too, hence the O(2k). The Python version claims to be O(k), so that's about right, and if I could avoid the reverse operation on the list, I'd be right alongside. 

Here's the actual 'double and add'.  

```scheme
;;; Add up the elements of the list
;;; to produce the "sum of the list"

(define (list+-iter l sum)
  (cond [(empty? l) sum]
        [else
         (list+-iter (cdr l) (+ sum (car l)))]))

(define (list+ l)
  (list+-iter l 0))

;;; create a sequence of numbers, starting
;;; with n, where successive members are the
;;; previous number in the list, multiplied by 2.
;;; The length of the list is determined by the bit list
;;; passed in, indicating the number of *2 items that are
;;; needed, since I need two lists of the same size to run a map
;;; (see below).

(define (multsof2 n bitlist mults)
  (cond [(empty? bitlist) (reverse mults)]
        [(empty? mults)
         (multsof2 n (cdr bitlist) (cons n mults))]
        [else
         (multsof2 n (cdr bitlist) (cons (* (car mults) 2) mults))]))

;;; Use the double and add method to multiply
;;; n and x

(define (dub_n_add n x)
  (let* ([b (bitz n)]
         [m (multsof2 x b '())])
    (list+
     (map (lambda (bit mult)
            (cond [(= bit 1) mult]
                  [else 0]))
          b
          m))))
```

I precompute a list of 2* multiples of the multiplier that is the same size as the list of bits that is created by the above code. That gives me two lists to run a 'map' on, at which point the actual map is easy - if the bit is set, then add the correct (multiplier * 2) chosen by its position in the list. Once the list is complete, sum it. 

Perhaps a better (more efficient) way would be to sum as I go, creating an accumulator, rather than post-processing the list. But that's for another day...

