# The Heartbreaking: Heartbleed Healing

_This text as originally written is in the format of response to [Sean Cassidy's post](http://blog.existentialize.com/diagnosis-of-the-openssl-heartbleed-bug.html) regarding [The Heartbleed Bug](http://heartbleed.com), ([CVE](http://cve.mitre.org)-[2014-0160](http://www.kb.cert.org/vuls/id/720951A)) as linked from [Ars Technica](http://arstechnica.com/security/2014/04/critical-crypto-bug-in-openssl-opens-two-thirds-of-the-web-to-eavesdropping).
It is a live document, and now resembles a FAQ. Changes are tracked by git:_

- _pertains to the catastrophic vulnerabilities in [OpenSSL](http://openssl.org)._
- It _(possibly) describes additional, yet-unpatched bug(s)._
- _It discusses strategies for "the heartbroken"._

## What is "heartbreaking"?

This whole thing is, really.

## What is "the Heartbreaking"?

A term I've just coined to name and describe:

- *The process by which the Internet will heal itself from The Heartbleed Bug, etc.*


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
Yet to be fully described, but to reproduce on Darwin (current Mac OS X + Xcode):

```sh
export CPPFLAGS=-flto
./Configure darwin64-x86_64-cc
make -j1; make -j1 test
./apps/openssl speed
```

At RC4, this will appear to loose the ability to count to three seconds.

No error reported. Occurs in OpenSSL 1.0.1g.

There are others but they seem more trivial. Like the "makedepend" thing.

To be expanded.

## You knew about this? Why didn't you report it?

I'll explain more, but basically, **yep**. Short answer: FIPS and resultant we-don't-take-patches-from-US-citizens-unless-you-preclear-them-with-the-US-gov't policy. Note that (from a user standpoint) I did report an "identical bug" in GMP. GMP catches it in its test suite and you can find my proposed workaround on the GMP mailing list.

To be expanded.

## Are you claiming to have had prior knowledge about Heartbleed?

No, definintely not. More like, I very strongly suspected "the Heartbreaking" was coming.

To be expanded.

## Could The Heartbleed Bug have been prevented?

Yes. IMO.

>1. Pay money for security audits of critical security infrastructure like OpenSSL. ... I would donate to this effort. Would you?

**No.** I am strongly of the opinion that this is how we got into this situation. This is not a problem that can be solved with dollars alone.

It is also why the FIPS program is fundamentally misguided: you cannot "certify" against unknown future attack vectors. Maybe if you have powers of supernatural prescience.

Instead, all it does is:
 - Give everyone a false sense of security; encourage complacence.
 - Divert time and resources.
 - Scream "exploit me".

Consider if this were _war_, not just _cyberwar_: step one: don't get shot. Step one of step one: do not paint target on back.

## I need to patch The Heartbleed Bug as of 9 A.M. yesterday.

Possibilities in the short-term:

- Rebuild with OpenSSL 1.0.1g (the 'patch'). I doubt this will be a permanent fix.
- Revert to OpenSSL 0.9.8.
- Use [CommonCrypto](http://www.opensource.apple.com/tarballs/CommonCrypto/CommonCrypto-60027.tar.gz) instead. Licensing is weird; but its a relatively easy patch that doesn't defer to current management. Not sure about portability.

## Are the decent alternatives to consider in the long-term?

I think so.

> Given how difficult it is to write safe C, I don't see any other options.  
> 3. Start writing alternatives in safer languages.

[Botan](http://botan.randombit.net) is the replacement for OpenSSL adopted in [BIND 10](http://bind10.isc.org). It is written in aggressive C++11.

_If unfamiliar, one may want to look into how STL performance guarantees work and compare the use of handwritten assembler in GnuTLS (via GMP) and OpenSSL (the BN library) for asymptotically fast operations._

Said with some naiveté (I could easily be missing something here), but, while not _written in C_, I don't see any glaring reason why this library shouldn't be able accommodate full *C ABI compatibility* given a little time and some quality code contribution.

Largely speculating, but in a standard scenario I this would more or less entail something like:

- Pull in the necessary STL headers and any needed non-header code.
- Write constructors/destructors so that it compiles with `-ffreestanding`, and/or
- Eliminating use of C++ RTTI and exception handling, or pull in libc++abi (with now-included libunwind).
- Write a C interface: take a look at the `llvm-c`/`clang-c` headers.


---

## WIP:

First public commit: 11 April 2014, ~12:00 P.M. UTC
Author: G. Nixon

---
I may come to having even posted this footnote?
This was under the "prior knowledge" paragraph:

 I also thought (and continue to assert) that my, effectively, screaming some "the-sky-is-falling" 'nonesense' about obscure bugs in OpenSSL when built with LLVM LTO / Intel IPO resulting in a weird benchmark, would have been received like a crazy bum on a street corner spouting prophecy. I have no security background/resume per se; I have insufficient technical knowlege to myself authoritatively pinpoint the issue at its core. Hence if I at all ppear to have "new information" here. If you want to say "BS", thats fine by me - its all in the past and quite irrelivant now. This is one (of others) reason I use an @aol.com email and have long since eliminated most social networks, etc. This is NOT a "claim", whatsoever, and NOT an invitation to "try me", but: it appears the only account I use regularly, or really care about, is, um, Github (sorta). Otherwise generally: AOL + OpenID, Apple ID, and one-off @grr.la email addresses. See my little script that generates fake-but-real name/phone/address combos from a statistically common names and a phone book. This is the full purpose of that script. If you don't believe me, please just go around saying I'm a POS liar trying to capitalize on a horrible situation, ather than, um, seeking to "punish" my non-"claim" here? I have no doubt you could do so, effortlessly, instantly. I don't even really use a particularly good passwords. That not my point, at all. I'm merely saying I had a hunch and tried to take precautions; and that it seems now, in hindsight, to possibly have been worth it. __To me.__

http://www.dailysabah.com/technology/2014/04/11/the-heartbleed-hit-list-the-passwords-you-need-to-change-right-now
https://www.hex-rays.com/products/ida/support/orderforms/floatingworld.pdf

Build a TOC:
* [What is "the Heartbreaking"?](#the-heartbreaking)
* [Why this term?](#bar)
* [License](#options)

---

## License
_License: Common sense; respect; customary copyright law._
_If **you** need it for some reason: you may use [CC BY SA 4.0]( http://creativecommons.org/licenses/by/4.0)._  
