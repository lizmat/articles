# PURL Support

> This is part 4 in the [SBOM](https://dev.to/lizmat/series/32933) series of blog posts

While working on the [SBOM::CycloneDX](https://raku.land/zef:lizmat/SBOM::CycloneDX) one of the fields caught my eye: [purl](https://cyclonedx.org/docs/1.6/json/#components_items_purl).  Initially I thought: "what does that have to do with our sister language"?  It quickly became clear that this was about a [**P**ackage **URL**](https://github.com/package-url/purl-spec?tab=readme-ov-file#purl).

## Deeper into the rabbit hole

From the [README](https://github.com/package-url/purl-spec?tab=readme-ov-file#context):

> A purl or package URL is an attempt to standardize existing approaches to reliably identify and locate software packages.
> A purl is a URL string used to identify and locate a software package in a mostly universal and uniform way across programming languages, package managers, packaging conventions, tools, APIs and databases.
> Such a package URL is useful to reliably reference the same software package using a simple and expressive syntax and conventions based on familiar URLs.

And the [list of adopters](https://github.com/package-url/purl-spec/blob/main/ADOPTERS.md) is already pretty impressive.

But, as there was no support for `PURL`s in Raku yet, it also meant support for it had to be created.  And as was the case with `SBOM::CycloneDX`, I decided to add support for the entire specification.  Which was also quite extensive, but not nearly as extensive as with `SBOM::CycloneDX`.

## A PURL module

Since I wanted to [SBOM::CycloneDX](https://raku.land/zef:lizmat/SBOM::CycloneDX) to be able to support the entire functionality of the CycloneDX 1.6 definition, I had no choice but to implement the whole of the PURL specification (and its supported types) as well.  Life can be hard when you're having fun!

At the moment of this writing, the PURL specification recognizes 31 types, each with their rules for handling and interpreting their native package specification into a Package URL.

Fortunately a pretty large test-set was provided for all registered types.  Which changed in format halfway through implementation, but that's what you get when you're at the bleeding edge of developments.

So there it was: [PURL support for Raku](https://raku.land/zef:lizmat/PURL).  Which allowed the [`purl` subset](https://raku.land/zef:lizmat/SBOM::CycloneDX#purl) of `SBOM::CycloneDx` to get validation.  For whatever SBOM thrown at it.

## Adding support for Raku module distributions

But implementing all of this, was not really of use for Raku module distributions itself.  Because there was **no** support for Raku defined in the PURL specification itself yet.

I decided to wing it.  Which was pretty easy, at first:
- type: "raku" (that was easy)
- namespace: that would be the "auth" of a Raku module distribution, e.g. "zef:raku-community-modules"
- name: that would be the name of the distribution, e.g. "MIME::Base64"
- version: that would be the version, e.g. "1.2.4"

So the Package URL (PURL) of the identity
```
MIME::Base64:ver<1.2.4>:auth<zef:raku-community-modules>
```
would be
```
pkg:raku/zef:raku-community-modules/MIME::Base6@1.2.4
```
Easy enough, right?

## Adding "raku" to the Package URL standard

But this was not enough of course: to make "raku" acceptable as a Package URL type, some work needed to be done.  And that work has been done in the ["Add Raku Programming Language as a purl-spec type](https://github.com/package-url/purl-spec/pull/550).  And there's hoping that this PR (with specification, documentation and tests) will be merged soon.

## Requirements versus Dependencies

But of course, there's always a catch.  If you look at the `META6.json` file of a Raku module distribution that needs other modules to be installed, you're generally looking at a "requirement". For instance, looking at the `META6.json` of [`Identity::Utils`](https://raku.land/zef:lizmat/Identity::Utils):
```
"depends": {
  "runtime": {
    "requires": [
      "String::Utils:ver<0.0.35+>:auth<zef:lizmat>"
    ]
  }
},
```
Notice the **+** in "0.0.35+".  This is Raku's way of indicating: "any version higher than or equal to 0.0.35 will be acceptable".  Such an identity string does **not** actually indicate a dependency, because it doesn't pin the actual version that will be used at runtime.

So how would this translate to a PURL?
```
use PURL;
say PURL.from-identity("String::Utils:ver<0.0.35+>:auth<zef:lizmat>");
# pkg:raku/zef:lizmat/String::Utils?vers=vers:raku/%3E=0.0.35
```
That doesn't look at all like "pkg:raku/zef:lizmat/String::Utils@0.0.35+"!  That's because PURLs use a more generic way to indicate version ranges, using the [**VE**rsion **R**ange **S**pecifier standard](https://github.com/package-url/vers-spec/blob/main/VERSION-RANGE-SPEC.rst#version-range-specifier).

Yet another rabbit hole to dive into.  But more about that in the next blog in this series.

## Conclusion
The [**P**ackage **URL** specification](https://github.com/package-url/purl-spec/blob/main/PURL-SPECIFICATION.rst#package-url-specification-v10x) attempts to reliably identify software packages.  Raku support for this standard was lacking in software, and in the standard.  The former has been addressed in the [`PURL`distribution ](https://raku.land/zef:lizmat/PURL), the latter is awaiting approval in a [Pull Request](https://github.com/package-url/purl-spec/pull/550).
