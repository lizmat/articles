# CycloneDX Support

> This is part 3 in the [SBOM](https://dev.to/lizmat/series/32933) series of blog posts

As there was no support yet in Raku for any of the SBOM standards, it was a question for which standard I should be developing: [SPDX 3.0.1](https://spdx.github.io/spdx-spec/v3.0.1/) or [CycloneDX 1.6](https://cyclonedx.org/docs/1.6/json).  For better or worse, I selected `CycloneDX` because it was recommended (thanks *Salve*!) and because it had better readable / browsable specification.  Because implementation of this would require a **lot** of reading and browsing!

## SBOM::CycloneDX

So by the end of June I started working on this, and the first version of the [SBOM::CycloneDX](https://raku.land/zef:lizmat/SBOM::CycloneDX) was uploaded on July 7th.  With the release `0.0.15` uploaded just the other day.

It turned out to be one of the largest single distributions I ever worked on: 125 classes (spread out over 51 files), 51 enums and 25 subsets, created by 5000+ lines of code and inline documentation, resulting in 4400+ lines of markdown documentation.

But why so many classes?  Is an SBOM just not really a hash of scalars, arrays and hashes?  Recursively?

Indeed it is.  But in order to be a valid SBOM, all sorts of restrictions apply to most fields, like an integer value higher than 0.  Or between 0 and 6. Or a numeric value between 0 and 1. And that's just numeric values.  It gets much more complicated for strings.

So what easier way would there be in Raku to have these constraints automatically checked for by `enum`, `subset`, and `class` constraints?

Also, it is nigh impossible to add custom functionality to specific parts of a hash.  And some custom functionality would be needed, and would allow much more flexibility with the creation of introspection tools.

So I opted for the creation of a class for each part of the CycloneDX 1.6 standard that was **not** a simple numerical or string value.  This also allowed for more internal re-use, as some classes are actually attributes of more than one other class.

I also wanted good error reporting on invalid SBOMs.  So instead of a single error in a part of the SBOM hash aborting the validation process, I wanted such errors to be collected and produced at the end (very much like [compile time worries](https://docs.raku.org/language/pragmas#worries) in the Raku Programming Language).

And as a bonus, this way I was able to put documentation of fields in the SBOM as [declarator docs](https://docs.raku.org/language/pod#index-entry-#=) in the source code, and generate the major part of the documentation from that. And that's how you wind up with 5000+ lines of code and inline documentation.

Although any Raku distribution will most likely only use a small subset of the functionality offered by the CycloneDX 1.6 standard, I decided to implement all of it, so that I wouldn't have to start adding stuff later.  And it will allow this module (and thus Raku) be used for **any** SBOM application outside of the Raku echo chamber.  Which would increase the chance of more people installing Raku, just to be able to used such an application.  That does not mean they need to know how to use Raku, or to program in the [Raku Programming Language](https://raku.org).

## enums

The "enum"-like values, as specified in the CycloneDX specification, posed an interesting problem.  The standard not only defines the strings, but also has some descriptions associated with them.  Descriptions I obviously wanted to keep around for documentation purposes.  And although we *can* specify declarator docs for an `enum`, it's (currently) impossible to specify a declarator doc for each of the possible values (and it would probably be impractical as well).

So I opted for the creation of a dedicated `Enumify` role that could be used on a class, thereby mimicking an actual `enum`, but which allows calling `.WHY` on the enum class, **and** on its instances (see [ENUMS API](https://raku.land/zef:lizmat/SBOM::CycloneDX#enums-api) for more information).  Which allows one to do:
```
use SBOM::enums <Phase>;
say "$_:\n$_.WHY()" with Phase<pre-build>;
# pre-build:
# BOM consisting of information obtained prior to a build process and
# may contain source files and development artifacts and manifests.
# The inventory may need to be resolved and retrieved prior to use.
```
And to make life easier for myself (and future maintainers) I've put all of the enum strings and descriptions as text files in the "resources" section of the distribution.  For instance, the `Phase` enum texts live in "resources/enums/Phase":
```
$ cat resources/enums/Phase
design
BOM produced early in the development lifecycle containing an inventory
of components and services that are proposed or planned to be used. The
inventory may need to be procured, retrieved, or resourced prior to use.

pre-build
BOM consisting of information obtained prior to a build process and
may contain source files and development artifacts and manifests.
The inventory may need to be resolved and retrieved prior to use.
...
```
A really simple format: first line is the name of the enum, and all lines until an empty line are the description.  All read in at compile time of the module, and then integrated into the `Phase` class.

## Scripts

The `SBOM::CycloneDX` installs a [`cyclonedx` script](https://raku.land/zef:lizmat/SBOM::CycloneDX#cyclonedx) that allows one to check the validity of an SBOM file, and/or do certain selections on them, and/or show the SBOM in normalized JSON, YAML or Raku code.  It probably will get more arguments, as people will find more uses for it.

> Actually, I'm hoping to be able to integrate SBOM capabilities into [App::Rak](https://raku.land/zef:lizmat/App::Rak) at some point in the not too distant future.

For example, if you would like to see the list of Package URLs used by the [SBOM-Raku distribution](https://raku.land/zef:lizmat/SBOM::Raku), one could do:
```
$ cyclonedx --purls SBOM-Raku/.META/SOURCE.cdx.json
pkg:raku/cpan:TIMOTIMO/JSON::Fast@0.19
pkg:raku/zef:demayl/Email::Valid@1.0.7
pkg:raku/zef:leont/YAMLish@0.1.2
pkg:raku/zef:lizmat/Identity::Utils@0.0.28
pkg:raku/zef:lizmat/PURL@0.0.14
pkg:raku/zef:lizmat/Rakudo::CORE::META@0.0.11
pkg:raku/zef:lizmat/SBOM::CycloneDX@0.0.13
pkg:raku/zef:lizmat/SBOM::Raku@0.0.10
pkg:raku/zef:lizmat/String::Utils@0.0.36
pkg:raku/zef:lizmat/VERS@0.0.2
pkg:raku/zef:raku-community-modules/MIME::Base64@1.2.4
pkg:raku/zef:raku-community-modules/OpenSSL@0.2.5
pkg:raku/zef:raku-community-modules/URI::Encode@1.0
pkg:raku/zef:rbt/Net::DNS@1.4
```
Which gives you a nice overview of all direct (*and transient*) dependencies of the distribution.

But what is a Package URL you say?   Well, that's for the next instalment of this series of blog posts!

## Conclusion
It was quite a lot of work to create a distribution that can handle the full extent of the [CycloneDX 1.6](https://cyclonedx.org/docs/1.6/json/) specification.  I'm quite sure bugs will be found, and it appears a 1.7 release of the CycloneDX specification is just around the corner (which would imply work to be done again on `SBOM::CycloneDX`).

But at least the groundwork has been laid to be able to handle SBOMs in the [Raku Programming Language](https://raku.org).
