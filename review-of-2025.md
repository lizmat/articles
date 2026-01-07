# Raku 2025 Review

> This is an updated version of the [Raku Advent Post](https://raku-advent.blog/2025/12/25/day-25-raku-2025-review/).

How time flies. Yet another year has flown by.

Let's first start with the technical stuff, as nerds do!

## Rakudo

Rakudo saw about 1650 commits (MoarVM, NQP, Rakudo, doc) this year, which is about 20% less than 2024. All of these repositories now have "main" as their default branch (rather than "master").

About 58% of the Rakudo commits were in the development of RakuAST (up from 33% in 2024), of which more than **95%** were done by *Stefan Seifert*. This work, that was supported by a [TPRF grant](https://news.perlfoundation.org/post/sseifert_rakuast_final), resulted in being able to build the part of Rakudo that's written in Raku using RakuAST (often referred to as the "**bootstrap**").

However there are still quite a few issues that need to be fixed before RakuAST-based Rakudo can be made the default. And thus be ready for the next language level release. Still, an amazing amount of work done by *Stefan Seifert*, so kudos!

### Under the hood, behind the scenes

*Geoffrey Broadwell* has done a major update of the internal MoarVM Unicode tools. Based on that work *Shimmerfairy* developed an update of the MoarVM Unicode support from 15.0 to 17.0. This includes support for some new emojis such as FINGERPRINT 🫆, SPLATTER 🫟, HARP 🪉. This means that all of the changes of [Unicode 16](https://www.unicode.org/versions/Unicode16.0.0/) and [Unicode 17](https://www.unicode.org/versions/Unicode17.0.0/) were implemented to a very high degree. Quite a feat!

*Patrick Böker* was **really** busy this year: new script runners not only made execution of CLI-scripts a bit faster, it also made it possible to run CLI-scripts on Windows without any issues.

Again, a lot of work was done on making the Continuous Integration testing produce fewer (and recently hardly any) false positives anymore. Which makes life for core developers a lot easier!

Very important for packagers: Rakudo now again has a reproducible build process, thanks for *Timo Paulssen* and others.

Also the REPL (Read, Evaluate, Print Loop) has been improved: grammar changes are now persistent, and it's also possible to enter multi-line comments. Some of these changes were backported from the [REPL module](https://raku.land/zef:lizmat/REPL), as described in this [blog post](https://dev.to/lizmat/repl-avalanche-45hh).

The tests for experimental Raku features (such as [:pack](https://docs.raku.org/language/experimental#pack), [:cached](https://docs.raku.org/language/experimental#cached), [:macros](https://docs.raku.org/language/experimental#macros)) have been moved from the Raku test-suite (roast) to the Rakudo repository, as they are technically not part of the definition of the Raku Programming Language.

Sadly, the JVM backend has hardly seen any updates the past year.  It was therefore decided to not mention the JVM backend in releases anymore, nor to make sure that there would not be any ecosystem breakage on the JVM backend before a release.

If you want the JVM to remain a viable backend, you are very much invited to get involved!

## New features in 6.d

The most notable new features in the default language level:

### Varargs support in NativeCall

After many years of people asking for this feature, *Patrick Böker* actually wrote the infrastructure in `MoarVM` and `Rakudo` to support calling `C`-functions that support a variable number of arguments using the [va_arg standard](https://linux.die.net/man/3/va_arg).

To give an example, it's now possible to create a `foo` subroutine that will call the `printf` `C`-function (from the standard library) that takes a format string as the first argument, and a variable number of arguments after that:
```raku
use NativeCall;
sub foo(str, **@ --> int32) is native is symbol('printf') {*}
foo "The answer: %d\n", 42;  # The answer: 42
```
The `printf()` `C`-function is a good example of using varargs.

### Support for pseudo-terminals (PTY)

Writing terminal applications has become much simpler with the support for pseudo-terminals that *Patrick Böker* has written.  This [Advent post](https://raku-advent.blog/2025/12/21/a-terminals-tale/) explains the how and why of this development, and the new [`Anolis` terminal emulator module](https://raku.land/zef:patrickb/Anolis).

Support is still a bit raw around the edges: pretty sure the coming year will see a lot of smoothing over, so that it can e.g. be used to create a full featured terminal-based Raku debugger!

### Hash.new(a => 42, b => 666)

A common beginner mistake was trying to create a new `Hash` (or `Map`) object by using `.new` and named arguments. This would silently create an empty `Hash` (or `Map`) because by default any unexpected named arguments are ignored in calls to `.new`. This was changed so that if `Hash.new` (or `Map.new`) is called with named arguments only, they will be interpreted as key/value pairs to be put into the `Hash` (or `Map`):
```raku
dd Hash.new(a => 42, b => 666);
# {:a(42), :b(666)}
dd Map.new(a => 42, b => 666);
# Map.new((:a(42),:b(666)))
```

### exits-ok

The `Test` module now also provides an `exits-ok` tester subroutine, making it a lot easier to test the exit behaviour of a piece of code.  It takes a `Callable`, and an integer value.  It expects the code to execute an [`exit` statement](https://docs.raku.org/routine/exit), and will then compare the (implicitly) given exit value with the given integer value.
```raku
use Test;
exits-ok { exit 1 }, 1;
# ok 1 - Was the exit code 1?
exits-ok { exit }, 1;
# not ok 2 - Was the exit code 1?
exits-ok { 42 }, 0;
# not ok 3 - Code did not exit, no exit value to check
```
> Note: after the Advent version of this post, it was decided to change the name from `exit-ok` to `exists-ok`, to make it more consistent with [`dies-ok`](https://docs.raku.org/routine/dies-ok) and [`lives-ok`](https://docs.raku.org/routine/lives-ok) tester subroutines.

## Language changes (6.e.PREVIEW)

The most notable additions to the future language level of the Raku Programming Language:

### Hash::Ordered

An implementation of ordered hashes (a hash in which the order of the keys is determined by order of addition) has become available:
```raku
use v6.e.PREVIEW;
my %h is Hash::Ordered = "a".."e" Z=> 1..5;
say %h.keys;    # [a b c d e]
say %h.values;  # (1 2 3 4 5)
```
This will probably get some easier syntactic sugar. Until then, the above syntax can be used.

## Language changes in RakuAST

The following changes are only seen when using `RakuAST` (by calling `raku` with the `RAKUDO_RAKUAST=1` environment variable set), and thus available by default when the next language level is released.

### Setting default language version

The `RAKU_LANGUAGE_VERSION` environment variable can be used to indicate the default language level with which to compile any Raku source code.  Note that this does **not** affect any explicit language versions specified in the code.
```
$ RAKUDO_RAKUAST=1 RAKU_LANGUAGE_VERSION=6.e.PREVIEW raku -e 'say nano'
1766430145418821670
$ RAKUDO_RAKUAST=1 RAKU_LANGUAGE_VERSION=6.e.PREVIEW raku -e 'use v6.d; say nano'
===SORRY!=== Error while compiling -e
Undeclared routine:
    nano used at line 1
```
Although of limited use while RakuAST is not yet the default, it **will** make it a lot easier to check the behaviour of code at different language levels, e.g. when running tests in the official test-suite (aka [roast](https://github.com/raku/roast?tab=readme-ov-file#the-official-raku-test-suite)).

### $?SOURCE, $?CHECKSUM

The compile-time variables `$?SOURCE` and `$?CHECKSUM` have been added.  The `$?SOURCE` compile-time variable contains the source of the current compilation unit.  If for some reason one doesn't want that to be included in the bytecode, then the `RAKUDO_OMIT_SOURCE` environment variable can be set.

The `$?CHECKSUM` compile-time variable contains a SHA1 digest of the source code.
```
$ RAKUDO_RAKUAST=1 raku -e 'say $?SOURCE'
say $?SOURCE
$ RAKUDO_RAKUAST=1 raku -e 'say $?CHECKSUM'
81892BA38B9BD6930380BD81DB948E4D7A9C14E7
$ RAKUDO_RAKUAST=1 RAKUDO_OMIT_SOURCE=1 raku -e 'say $?SOURCE'
Nil
```
These additions are intended to be used by the `MoarVM` runtime debugger, as well as by packagers for verification.

### Localization

Most of the localization work has been removed from the Rakudo core and put into separate [`Raku-L10N` project](https://github.com/raku-L10N).  And there it gained a few new contributors!  For a progress report, checkout out *habere-et-dipertire*'s advent blog post titled [Hallo, Wêreld!](https://raku-advent.blog/2025/12/18/day-18-hallo-wereld/)

To make it easier to work with code in a specific localization, each localization comes with a "fun" command line script, and an official one.  So for instance, the [Dutch localization](https://raku.land/zef:l10n/L10N::NL) has a "dutku" (for **dut**ch ra**ku**) executable (well, actually a CLI script), but also a "kaas" one.  Same for French ("freku" and "brie"), etc.  The fun one is usually associated with a favourite foodstuff of the language in question.
```raku
$ dutku -e 'zeg "foo"'
foo
$ kaas -e 'zeg "foo"'
foo
$ freku -e 'dis "foo"'
foo
$ brie -e 'dis "foo"'
foo
```

### RakuDoc

The RakuDoc v2.0 specification was completed in December 2024, and 2025 was spent implementing it. A compliant renderer is now available by installing the [`Rakuast::RakuDoc::Render`](https://raku.land/zef:finanalyst/Rakuast::RakuDoc::Render) distribution. Work then began on a document management system called [`Elucid8`](https://raku.land/zef:finanalyst/Elucid8::Build) which renders the whole Raku documentation ([development preview](https://dev.docs.raku.org)).

From September onwards, Damian Conway and Richard Hainsworth worked on a enumeration system (originally envisioned for RakuDoc v3.0) so that any block - meaning any paragraph, heading, code snippet, formula, etc - can be enumerated simply by prefixing the blocktype with num. It is a far more flexible system than anything encountered in the editor space.

The RakuDoc specification is now at [version 2.20.2](https://htmlpreview.github.io/?https://github.com/Raku/RakuDoc-GAMMA/blob/numalias-clarification/rakudoc_v2.html).  The new enumeration specification has not yet been merged to main because work is still being done on getting `Rakuast::RakuDoc::Render` to implement the new standard.

It really looks like RakuDoc has the potential to becoming **the** markup language for any type of serious documentation, and a direct competitor to markdown.

## Ecosystem

The [Raku ecosystem](https://raku.land) has seen quite a lot of developments.  Many modules got moved to the [Raku Community Modules Adoption Center](https://github.com/raku-community-modules) where they got a [new lease on life](https://raku.land/zef:raku-community-modules).  But there were also quite a few new modules, and interesting updates to existing modules in 2025.

### Statistics

According to [raku.land](https://raku.land/stats) in 2025, *508* Raku modules have been updated (or first released): up from *367* in 2024 (an increase of 38%). There are now *2435* different modules installable by [`zef`](https://raku.land/zef:ugexe/zef) by just mentioning their name. And there are now 13843 different versions of Raku modules available from the Raku Ecosystem Archive, up from 12181 in 2024, which means more than 4.5 module updates / day on average in 2025 (up from 3.9 updates / day).

### Interesting new modules

The modules that yours truly found interesting, so a very personal list!  In alphabetical order:

- [`AI::Gator`](https://raku.land/zef:bduggan/AI::Gator) - AI Generic Assistant with a Tool-Oriented REPL.
- [`Air`](https://raku.land/zef:librasteve/Air) - Just build websites the right way.  See also [We’re Walking On The Air](https://raku-advent.blog/2025/12/20/day-20-were-walking-on-the-air/).
- [`Anolis`](https://raku.land/zef:patrickb/Anolis) - A Terminal Emulator.  See also [A Terminal’s Tale](https://raku-advent.blog/2025/12/21/a-terminals-tale/).
- [`ASTQuery`](https://raku.land/zef:FCO/ASTQuery) - Query and manipulate Raku’s Abstract Syntax Trees (RakuAST) with an expressive syntax.  See also [From ASTs to RakuAST to ASTQuery](https://dev.to/fco/from-asts-to-rakuast-to-astquery-c3f).
- [`Cromponent`](https://raku.land/zef:FCO/Cromponent) - A way create web components with cro templates.  See also [Cromponent new features](https://dev.to/fco/cromponent-new-features-3bhf).
- [`DataStar`](https://raku.land/zef:arunvickram/DataStar) - A Raku SDK for the data-star hyper media framework. See also [Raku To The Stars](https://raku-advent.blog/2025/12/11/day-11-raku-to-the-stars/).
- [`Draku`](https://raku.land/zef:bduggan/Draku) - A documentation browser for Raku.
- [`Elucid8::Build`](https://raku.land/zef:finanalyst/Elucid8::Build) - Renders RakuDoc sources in multiple languages to web site.  See also [Create a minimal site with Elucid8](https://dev.to/finanalyst/create-a-minimal-site-with-elucid8-1gf8).
- [`Gnome::Gtk4`](https://raku.land/zef:martimm/Gnome::Gtk4) - The language binding to GNOME’s user interface toolkit version 4.  See also [Tools for Gnome::Gtk4](https://raku-advent.blog/2025/12/05/day-5-tools-for-gnomegtk4/).
- [`LLM::Graph`](https://raku.land/zef:antononcube/LLM::Graph) - Efficiently schedule and combine multiple LLM generation steps.  See also [Robust code generation combining grammars and LLMs](https://raku-advent.blog/2025/12/06/day-6-robust-code-generation-combining-grammars-and-llms/).
- [`Math::NumberTheory`](https://raku.land/zef:antononcube/Math::NumberTheory) - Raku package with Number theory functions.  See also [Numerically 2026 Is Unremarkable Yet Happy](https://raku-advent.blog/2025/12/22/day-22-numerically-2026-is-unremarkable-yet-happy/).
- [`SBOM::Raku`](https://raku.land/zef:lizmat/SBOM::Raku) - Raku specific SBOM functionality.  See also [Towards more accountability of Raku programs](https://dev.to/lizmat/towards-more-accountability-of-raku-programs-3g2).
- [`Test::Coverage`](https://raku.land/zef:lizmat/Test::Coverage) - Check test files for sufficient coverage.  See also [Towards more coverage](https://dev.to/lizmat/towards-more-coverage-fne).
- [`Text::Emoji`](https://raku.land/zef:lizmat/Text::Emoji) - Provide :text: to emoji translation
- [`Zeco`](https://raku.land/zef:tony-o/Zeco) - An ecosystem hosting module for raku.  See also [Grant Report: Raku Ecosystem Final](https://news.perlfoundation.org/post/raku-ecosystem-tonyo-final).

### Modules with notable updates

Also in alphabetical order:

- [`App::Rak`](https://raku.land/zef:lizmat/App::Rak) - 21st century grep / find / ack / ag / rg on steroids
- [`Cro`](https://raku.land/zef:cro/cro) - cro command line and web tool
- [`PDF`](https://raku.land/zef:dwarring/PDF) - Base classes for reading, manipulation and writing of PDF data
- [`Red`](https://raku.land/zef:FCO/Red) - A Raku ORM
- [`REPL`](https://raku.land/zef:lizmat/REPL) - A more easily configurable REPL.  See also [REPL Avalanche](https://dev.to/lizmat/repl-avalanche-45hh).
- [`Rakuast::Rakudoc::Renderer`](https://raku.land/zef:finanalyst/Rakuast::RakuDoc::Render) - renders RakuDoc v2 to text, HTML, HTML-Extra, Markdown.
- [`Slang::Nogil`](https://raku.land/zef:lizmat/Slang::Nogil) - allow sigilless scalar variables.  See also [Allowing for fewer dollars](https://raku-advent.blog/2025/12/07/day-7-allowing-for-fewer-dollars/).
- [`Terminal::LineEditor`](https://raku.land/zef:japhb/Terminal::LineEditor) - Generalized terminal line editing
- [`zef`](https://raku.land/zef:ugexe/zef) - Raku Module Management

## Bots

A new experimental bot has appeared on the #raku-dev IRC channel: `rakkable`.  It is basically an interactive front end for the [new "rakudo-xxx"](https://raku.land/zef:lizmat/App::Rak/changes?v=0.3.19) features of [`App::Rak`](https://raku.land/zef:lizmat/App::Rak).  Which in turn is based on the new [`Ecosystem::Cache`](https://raku.land/zef:lizmat/Ecosystem::Cache) module.  This allows easy searching in all **most current** versions of modules in the ecosystem.

For instance: look in the Raku ecosystem for code mentioned in "provides" sections that contain the string "Lock.new" and which **also** have the string "$!lock":
```
<lizmat> rakkable: eco-provides Lock.new --and=$!lock
<rakkable> Running: eco-provides Lock.new --and=$!lock, please be patient!
<rakkable> Found 30 lines in 25 files (24 distributions):
<rakkable> https://gist.github.com/fa2424aebf085ea656b436c63722bf9d
```
The bot currently only lives on the #raku-dev, but can also be accessed directly without needing the "rakkable:" prefix.

## Non-technical stuff

Yes, there's also non-technical stuff in the Raku world!

### Websites

The [raku.org](https://raku.org) website has been completely renewed, thanks to *Steve Roe* who has taken that on.  It's now completely dogfooded: hypered with [htmx](https://htmx.org/). Aloft on [Åir](https://harcstack.org/). Constructed in [cro](https://cro.raku.org/). Written in [raku](https://raku.org/).  &  Styled by [picocss](https://picocss.com/).

### Documentation

The [Raku Documentation Project](https://github.com/raku/doc?tab=readme-ov-file#official-documentation-of-raku) has gained quite a few collaborators, who are working on making the [Raku documentation](https://docs.raku.org) more accessible to new users. One factor making this easier, is that the CI testing for the documentation has become about 4x as fast by using RakuAST RakuDoc parsing.

### Social Media

Yours truly stopped using what is now X (formerly Twitter).  It hurt.  But [Bluesky](https://bsky.app) and [Mastodon](https://mastodon.social/explore) are good alternatives, and the people important to yours truly have moved to them.  If you haven't yet, you probably should.  As well as the people important to you.

Please be sure to mention the **#rakulang** tag when posting about the [Raku Programming Language](https://raku.org)!

### Conference / Core Summit

Sadly it has turned out to be impossible to organize a Raku Conference (neither in-person or online) this year.  Hoping for better times next year!  It just really depends on people wanting to put in the effort!

It **was** possible to organize a [second Raku Core Summit](https://dev.to/patrickbkr/the-second-raku-core-summit-3d2) in 2025!

### Weekly

The {Rakudo Weekly News](https://rakudoweekly.blog/blog-feed/) has been brought to you by *Steve Roe* in the 2nd half of 2025 (and for the foreseeable future).  With some new features, such as code gists!  Kudos!

### Problem Solving

The [Problem Solving repository](https://github.com/Raku/problem-solving?tab=readme-ov-file#-problem-solving) has seen an influx of [**36** new issues](https://github.com/Raku/problem-solving/issues?q=is%3Aissue%20state%3Aopen%20created%3A%3E2025-01-01%20created%3A%3C%3D2025-12-31) in 2025. They [**all**](https://github.com/Raku/problem-solving/issues) deserve your attention and your feedback! Some of them specifically ask for your ideas, such as:

- [Further Improvements to the Raku Website Landscape](https://github.com/Raku/problem-solving/issues/502)
- [Interest & Engage the Next 1000 Raku coders](https://github.com/Raku/problem-solving/issues/507)
- [Raku / Rakudo adapt and adopt the LLVM AI Tool Policy](https://github.com/Raku/problem-solving/issues/510)
- [Preparing module META6.json information for the future](https://github.com/Raku/problem-solving/issues/491)
- [Raku Classification System](https://github.com/Raku/problem-solving/issues/489)

Please, don't be shy and have your voice heard!

### Raku Steering Council

Sadly, *Vadim Belman* and *Stefan Seifert* have indicated that they wanted to step down from the Raku Steering Council. They are thanked for all that they have done for Raku, the Raku Community in general, and the Raku Steering Council in particular.

*John Haltiwanger* has accepted an invitation to join the Raku Steering Council. The seat opened by *Stefan Seifert* will not be filled for at least the coming 6 months.

### The Raku Foundation

After writing a [blog post about a Raku Foundation](https://dev.to/lizmat/towards-a-raku-foundation-3ne2), a [problem solving issue](https://github.com/Raku/problem-solving/issues/477) and many discussions at the second Raku Core Summit, there finally is a version of the [Raku Foundation Documents](https://lizmat.github.io/Raku-Foundation-Documents/) that everybody can agree on (at least for now).

Yours truly urges the readers to check out [The Articles Of Association](https://lizmat.github.io/Raku-Foundation-Documents/articles-of-association.html), as these will be very hard to change once the foundation is established.

If you are really interested in this kind of stuff, please check out the [Regulations for the operation of the Raku Foundation](https://lizmat.github.io/Raku-Foundation-Documents/RakuFoundationRegulations.html) as well.

And if you would like to become part of the initial [Executive Board](https://lizmat.github.io/Raku-Foundation-Documents/articles-of-association.html#The_Exec) or the [Supervisory Board](https://lizmat.github.io/Raku-Foundation-Documents/articles-of-association.html#Super), please send an email to [founding@raku.foundation](founding@raku.foundation).

## Summary

Looking back, again an amazing amount of work has been done in 2025!  And not only on the technical side of things!

Hopefully you will all be able to enjoy the Holiday Season with sufficient R&R. Especially [*Kane Valentine*](https://irclogs.raku.org/raku/gist.html?,2024-04-23Z12:47,2024-04-23Z12:48,2024-04-23Z12:50-0001,2024-04-23Z12:52,2024-04-23Z13:14,2024-04-23Z14:34) (aka kawaii) who is still going strong in their new role:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5yfm8odvtjzghjz2lxd7.png)

And on that note: Слава Україні!  Героям слава!
