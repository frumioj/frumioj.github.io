I've been exploring lots of languages lately. Scheme (again). Rust. Swift. They all bring something to the table. Frankly, I love functional languages, so Scheme is the language for me, if I get to choose. 

But when it comes to "systems programming", I am still most comfortable in C.

I've been trying to understand how secure enclaves work, and how they differ from TPMs and HSMs. With a lovely new Apple M1 system in hand, I've been looking at the secure enclave APIs on MacOS/iOS.

I started by looking at the Apple documentation. It's sparse, and seemingly unwilling to give more than the just the one example use. Should I be using the CryptoKit API, or the lower-level APIs? There are hints I should use CryptoKit, but few complete examples to follow. All of the examples are in Swift, a language I don't know. They all assume XCode, an IDE I have never really used. 

How hard can it be though?

I try to import an XCode project. 

It is out of date, and XCode tells me that I have two choices. 

Install XCode 10 (a MUCH earlier version of XCode than I have!) and perform an official language migration, or I can update the project language to Swift 5 in my current XCode, and expect a flurry of compile errors that I probably won't understand. The former sounds terrifying (go backwards from XCode 12 - XCode 10, migrate the project, and then go forwards in time again to XCode 12, with presumably each of these installations being multiple GB downloads, and who knows what errors occuring as a result of installing a legacy IDE version on my brand-new Mac). The latter sounds terrifying too, but the unknowns of fixing Swift language upgrade bugs _feel_ more unknown to me, and thus potentially easier to deal with. So I change the project language to Swift 5. 

Surprisingly, with some decent prompting from XCode itself as to the cause of the errors, I am able to fix the compiler errors and even some of the warnings.

The project builds, 
