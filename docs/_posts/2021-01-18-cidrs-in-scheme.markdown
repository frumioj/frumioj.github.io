---
layout: post
title: "Dealing with CIDRs in Scheme"
date: 2021-01-18 12:00:00 -0000
categories: scheme networking racket
---
I'm always forgetting how to convert a [CIDR] into a netmask. I remembered
the part about (32 - the CIDR) telling you the important part. But I
forgot that the important part it tells you is that this gives you the power of 2, and
you still need to do some work to get from that to the number of addresses, and the subnet mask you need.

Here is that work!

```scheme

> (define (cidr slash)
    (let ([power-of-two (- 32 slash)])
      (format "~a addresses, subnet mask: 255.255.255.~a" (expt 2 power-of-two) (- 255 (- (expt 2 power-of-two) 1)))))

> (cidr 26)
"64 addresses, subnet mask: 255.255.255.192"

```

[CIDR] https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
