---
layout: post
title: "My coding experience"
date: 2022-03-02 12:00:00 -0000
categories: thingsiforget
---

Whenever I feel most like a tech "imposter" (usually right before, or after a technical interview), it helps to remember just how many 
different things I've actually coded.

Here is a partial list of my significant coding contributions:

* Mainframe Connect (1991/1992)

A Clipper/dBase UI running in DOS on x86 that screen-scraped an IBM 3270 mainframe financial controller, trapped function key interrupts in x86 assembly, and called the lu6.2 C library to interact with the financial controller/mainframe. This was intended to allow users to work with the mainframe systems, while drawing a more user-friendly UI, and allowing the use of function keys to provide non-mainframe actions, like context-sensitive help. This application was deployed to 550 bank branch offices with 5+ PCs in each branch. I received a special letter of commendation from the company board for this work, as it significantly improved the ability of our branch employees to do their jobs.

_Programming languages:_ **Clipper/dBase, C, x86 assembly**

Performance constraints: DOS 640k memory limit, use of assembly language to access keyboard interrupts, screen-scraping mainframe screen via lu 6.2 C library.

* Mortgage APR/compound interest calculator (1991/1992)

An application that created mortgage quotations for bank customers, using a successive approximations algorithm [related explanation](https://www.mtgprofessor.com/formulas.htm). This cut the mortgage quotation/application timeframe from 2+ days to 20 minutes, and could be conducted fully with the customer while in the office.

_Programming languages:_ **Clipper/dBase, C**

Performance constraints: follow legally-approved algorithms for calculating APR/compound interest, fastest-possible calculation given commodity PC hardware, DOS 640k memory limit.

* [Clickshare](https://www.clickshare.com) micropayments system (1995-1996)

A multi-party micropayments system for allowing small payments for newspaper articles.

_Programming languages_: **C**

*  Employease Network (Acquired by ADP, and now called [https://workforcenow.adp.com/](https://workforcenow.adp.com/) employee benefits system (1997-2000)

One of the very first "software as a service" applications (1997), Employease offered a browser-based interface for managing employee benefits, targeted at insurance brokers, but also providing the very first employee "self-service" interface for managing their own benefits. I was employee #10, and wrote much of the early code for the self-service system. This was originally written in the Netscape Livewire server-side Javascript language, and later replaced by Java servlets.

_Programming languages:_ **Javascript, Java**

Performance constraints: latency of early Internet systems.

* Serene: [a web services framework](https://github.com/SymbianSource/oss.FCL.sf.mw.websrv/tree/master/webservices)

A framework for the Series 60 (Symbian) mobile phone, implementing support for the Liberty ID-WSF and Microsoft/IBM WS-STAR SOAP/XML specifications to allow the use of mobile-phone based privacy-preserving web services. This was deployed most recently to 3 million Nokia S60 phones as the basis for the Microsoft Windows Live Messenger Symbian port. I then ported portions of this also to Linux/D-BUS in C.

_Programming languages:_ **Java, C, Symbian C++**

Performance constraints: memory on mobile phones, latency in early mobile networks, poor mobile support for cryptography algorithms.

* Object capability-based web browser

Webkit-based multi-process web browser that separated memory per-URL for all internet resources (each URL handled by a separate OS process)

_Programming languages:_ **C**

Performance constraints: separation of memory vs. collaboration between processes

* OSH - Ovi "Shell"

A terminal shell for using the Nokia/HERE web services (e.g. mapping, routing, "tweeting") from a CLI environment

_Programming languages:_ **Javascript**

* Implementing an OAuth 2 based system for authorizing access to Nokia/HERE APIs

Wrote Java servlet-based system for implementing OAuth access to our APIs (https://developer.here.com)

_Programming languages:_ **Java**

* Implementing W3C timed-text captions in [ccextractor](https://github.com/frumioj/ccextractor)

Added support for W3C Timed Text to the ccextractor open source project, on behalf of ESPN (which needed this for Windows).

_Programming languages:_ **C**

* Lilith - a system for delivering video chunks out of order to better utilize massive bandwidth

Described for the [Erlang Factory conference](http://www.erlang-factory.com/conference/SFBay2013/speakers/JohnKemp) System used a Javascript client, websocket connections to a server which ran Erlang, and the Dynamo DB-based riak-core library to store and deliver video chunks.

_Programming languages:_ **Erlang, Javascript**

* Crypto Services

I was the lead engineer creating the prototypes for a suite of cryptographic services, and later led the team doing the development:

i) Java, Python and go-lang clients for a hardware key management system (utilizing an XML-based protocol)

_Programming languages:_ **Java, Python, go-lang**

ii) Simple-signing service - a service for signing XML and JSON data using keys stored in hardware security modules (HSMs) and using PKCS11 APIs. This involved modifying two open-source libraries to add support for HSM-resident keys (xmlsec, latchset/jose) and maintaining internal forks of these repositories.

_Programming languages:_ **go-lang, C**

12. Cosmos SDK/Keystone [low-s normalization](https://github.com/cosmos/cosmos-sdk/pull/9738), [Adding secp256k1 to pkcs11 lib](https://github.com/frumioj/crypto11), [example usage of capnproto](https://github.com/capnproto/go-capnproto2/pull/204) and [Keystone key-management for Cosmos](https://github.com/frumioj/keystone) for some recent contributions)

Implementing a distributed key management system for the Cosmos SDK - an SDK used in the creation of a number of blockchain-based systems, utilizing HSMs.

_Programming languages:_ **go-lang**
