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

I have no idea of how this compares to the Python generator approach, but seems to be O(2k) where k is the number of bit-shift operations on n, or the length of the list produced. Reversing the list is O(k) too, hence the O(2k). 



