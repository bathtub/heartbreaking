# The Heartbreaking: Heartbleed Healing

_This text as originally written is in the format of response to [Sean Cassidy's post](http://blog.existentialize.com/diagnosis-of-the-openssl-heartbleed-bug.html) regarding [The Heartbleed Bug](http://heartbleed.com),  
([CVE](http://cve.mitre.org)-[2014-0160](http://www.kb.cert.org/vuls/id/720951A)) as linked from [Ars Technica](http://arstechnica.com/security/2014/04/critical-crypto-bug-in-openssl-opens-two-thirds-of-the-web-to-eavesdropping).
It is a live document tracked by git, and now resembles a FAQ._

- It _pertains to the catastrophic vulnerabilities in [OpenSSL](http://openssl.org)._
- It _(possibly) describes additional, yet-unpatched bug(s)._
- It _discusses strategies for "the heartbroken"._

## What is "heartbreaking"?

This whole thing is, really.

## What is "the Heartbreaking"?

A term I've just coined to name and describe:

- *The process by which the Internet will heal itself from The Heartbleed Bug, etc. Over the years.* 


- Could also be put it: *the process of identifying, testing, and/or replacing every piece of "secure" software presently in deployment that possibly links OpenSSL 1.0.1[a-f].*

Note OpenSSL by default builds and links as a *static* library unless explicitly configured to build a shared/dynamic library instead. A telling symbols or string may, or, may not, be present in a linked image/symbol table, depending on configuration. A very short and incomplete list of software which may include this code non-obviously:  

 - Python
 - Ruby
 - Node.js
 - curl
 - ...

## Why coin _another_ term? Why _this_ term?

- Because the bug called "The Heartbleed Bug" is purportedly 'patched', but
  the scale of this issue is so massive that the term **patch** does not apply.
- Because I believe there are more Heartbleed **bugs**.

- To prevent accidentally calling "The Heartbleed Bug" the _Heartbreak_ Bug, by forcing a distinction (this is something I caught myself accidentally doing).

## Another bug?
Yes. Its definitely a bug, though I'm not sure of its root cause, and I don't know its severity or if its exploitable. ~~It may be like a bug I found in GMP last year; it may be related to Heartbleed; it may be something else altogether.~~

**Edit: I now just realized (whatever _this_ bug is), that it affects OpenSSL 0.9.8+ as well. So, it isn't Heartbleed. If we're lucky, it could just be a bug in the built-in benchmarking functionality.**

Summary: when built with LTO/IPO (LLVM or Intel), OpenSSL will compile and pass 'make test'. However, if you then invoke `openssl speed`, it will soon loose its ability to count to three.

I believe this should be enough to reproduce:
(Code is for Darwin; I'm 99% confident this is not platform specific.)

```sh
export CPPFLAGS=-flto
./Configure darwin64-x86_64-cc
make -j1; make -j1 test
./apps/openssl speed
```

At RC4, this will appear to loose the ability to count to three seconds.
No error reported. Occurs in OpenSSL 1.0.1g.

There are others; but they seem more trivial. Like "makedepend" and "domd".

_To be expanded._

## You knew about this? Why didn't you report it?

Yes. Because of the OpenSSL project's decision to participate in US FIPS 140-2 product validation (see below), they have a standing policy to not accept patches from US citizens, unless registered and pre-cleared first with the United States government. That is something I am absolutely unwilling to do, so I just sat on it.

Note that (from a user standpoint, not a technical one) I did report an "identical bug" in GMP. GMP catches this in its test suite; and you can find my proposed workaround on the GMP mailing list.

_To be expanded._

## Are you claiming you knew about Heartbleed?

No, definitely not. More like, I very strongly suspected "the Heartbreaking" was coming, that's all.

_To be expanded._

## Could The Heartbleed Bug have been prevented?

Yes.

>1. Pay money for security audits of critical security infrastructure like OpenSSL. ... I would donate to this effort. Would you?

**No.**
I am strongly of the opinion that this is how we got into this situation in the first place.
This is not a problem that can be solved by throwing dollars at it.

_Audit your deployment, in its environment. Not the software itself, in a clean room._

This is why the FIPS 140-2 validation program is fundamentally misguided: one cannot "certify" against unknown, future attack vectors. Maybe if you have powers of supernatural prescience. Instead, all it does is:
 - Give everyone a false sense of security; encourage complacence.
 - Divert time and resources.
 - Screams "exploit me". The first step to winning a war: do not get shot and killed. The first step to not getting shot: do not paint target on back.

Edit:  
**This is same point made (much more saliently and eloquently, and with the qualifications to do so) [by Steve Marquess](http://veridicalsystems.com/blog/secure-or-compliant-pick-one). _Required reading_.**  

_The OpenSSL FIPS module itself is not affected by the Heartbleed bug._

## I need to patch The Heartbleed Bug as of 9 A.M. yesterday.

Possibilities in the short-term:

- Rebuild with OpenSSL 1.0.1g (the 'patch'), or revert to OpenSSL pre-1.0.1. I doubt this will be a permanent fix.
- On Darwin, use [CommonCrypto](http://www.opensource.apple.com/tarballs/CommonCrypto/CommonCrypto-60027.tar.gz) instead. Not portable.

## Are the decent alternatives to consider in the long-term?

> Given how difficult it is to write safe C, I don't see any other options.  
> 3. Start writing alternatives in safer languages.

#### Botan
[Botan](http://botan.randombit.net) is the replacement for OpenSSL adopted in [BIND 10](http://bind10.isc.org). It is written in C++11.

_If unfamiliar, one may want to look into how STL performance guarantees work and compare the use of handwritten assembler in GnuTLS (via GMP) and OpenSSL (the BN library) for asymptotically fast operations._

Said with some naiveté (I could easily be missing something here), but, while not _written in C_, I don't see any glaring reason why this library shouldn't be able accommodate full *C ABI compatibility* given a little time and some quality code contribution.

Largely speculating, but in a standard scenario I this would more or less entail something like:

- Pull in the necessary STL headers and any needed non-header code.
- Write constructors/destructors so that it compiles with `-ffreestanding`, and/or
- Eliminating use of C++ RTTI and exception handling, or pull in libc++abi (with now-included libunwind).
- Write a C interface: take a look at the `llvm-c`/`clang-c` headers.

#### LibTomCrypt
I've yet to actually try it myself. Not sure about speed. But, public domain!

#### GnuTLS
If your project allows for LGPL3 or GPL2. GMP (starting with version 6.0.0) now once again allows GPL **2** licensing, and no longer forces LGPL3.

Other (full) GPL2 libraries:
#### CyaSSL
#### PolarSSL

===
_Author: G. Nixon_  
_First public commit: 11 April 2014, ~12:00 P.M. UTC_  
_License: Common sense; respect; customary copyright law._ _If **you** need it for some reason: you may use [CC BY SA 4.0]( http://creativecommons.org/licenses/by/4.0)._  
