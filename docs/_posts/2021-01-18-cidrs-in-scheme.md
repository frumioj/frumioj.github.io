layout: post
title: "Dealing with CIDRs in Scheme"
date: 2021-01-18 12:00:00 -0000
categories: scheme networking racket

I'm always forgetting how to convert a CIDR into a netmask. I remembed
the part about 32 - the CIDR telling you the important part. But I
forgot that the important part it tells you is the power of 2, and
then you still need to do some work. Here is that work!

```lisp
(define (cidr slash)                                                                                                                                               (let ([power-of-two (- 32 slash)])                                                                                                                                 (values (expt 2 power-of-two) (- 255 (- (expt 2 power-of-two) 1)))))
```
