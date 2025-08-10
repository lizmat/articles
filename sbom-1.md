# Towards more accountability of Raku programs

Shortly after the [Second Raku Core Summit](https://dev.to/lizmat/the-second-raku-core-summit-f99) in June, it became clear to me that there had been one elephant in the room that hadn't been discussed enough: the gradual enforcement of the [Cyber Resilience Act](https://en.wikipedia.org/wiki/Cyber_Resilience_Act) in Europe. And how that would affect Open Source, and the [Raku Programming Language](htts://raku.org) specifically.

## CRA effects

In short, without getting into the deep end immediately, it all boils down to this statement:

> Companies need to conduct cyber risk assessments before a product is put on the market and retain its data inventory and documentation throughout the 10 years after being put on market or its support period, whichever is longer.

The crux for programming languages in general, and thus for Raku, lies in the "cyber risk assessments" that a company would need to conduct. Doing a cyber risk assessment on a Raku program and all of its dependencies is currently a task that would have to be done "manually", as it were. And that would just be too complex, and too costly for many companies to be doing for open source projects. And thus be a reason for companies to stop using open source.

## SBOMs

Fortunately the concept of [Software Bill Of Materials](https://en.wikipedia.org/wiki/Software_supply_chain) was introduced in the 2010s. And it has become a requirement that every trustworthy software producer should provide. And that includes the Raku Programming Language.  And all of the programs that have been, are and will be made with it.

As with any great ideas for standardization, there is more than one standard: there are basically two flavours of SBOMs in use these days: the [System Package Data eXchange (SPDX)](https://spdx.dev) and [CycloneDX, based on ECMA-424](https://cyclonedx.org). The latter appears to be more suited for open source projects such as the Raku Programming Language.

## What needs to be produced?

In the end, a machine-readable representation of all dependencies of an application that is installed in a given environment, let's say an SBOM. Because that would allow a programatically driven assessment of any security threats in light of a given vulnerability in a known component's version.

To give an example: suppose a vulnerability in the [OpenSSL library](https://openssl-library.org) is discovered. And a company doing business in Europe is using a [Cro](https://cro.raku.org) application as backend for its mobile app. Several parts of Cro depend on the [IO::Socket::Async::SSL](https://raku.land/zef:raku-community-modules/IO::Socket::Async::SSL) distribution, which in turn depends on the [OpenSSL](https://raku.land/zef:raku-community-modules/OpenSSL) distribution. Which in turn depends on the OpenSSL library.

That company would need to do a security assessment, and if found to be also vulnerable, do the appropriate updates to ensure conformance with the CRA rules and laws. And do that within a quite restricted timeframe in order to avoid penalties. And then produce an updated SBOM to certify that it is in conformance.

## Are we there yet?

No. Discussions in the EU are still going on about what would need to be produced and when. But at this point in time, it seems to be more like *months* before conclusions are reached, rather than years. It **is** time to prepare the Raku Programming Language for this new environment.

So, is the CRA a threat? Or an opportunity? I'd say it is an opportunity for Open Source projects in general, and the Raku Programming Language in particular, disguised as a threat.

## Conclusion

Exciting times are ahead. In the following blog posts I will go deeper in the aspects of the CRA, and the software that I have developed so far to help in turning the threat of the CRA into an opportunity for the Raku Programming Language.
