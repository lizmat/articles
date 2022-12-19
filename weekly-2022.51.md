# 2022.51 Hijacking D3

*Anton Antonov* has published another video, this time [about an excellent new module Javascript::D3](https://www.youtube.com/watch?v=YIhx3FBWayo) to create beautiful graphs with Raku in Jupyter notebooks, or just even in HTML.  It also comes with a (blog post](https://rakuforprediction.wordpress.com/2022/12/15/javascriptd3/) and comments on [/r/rakulang](https://www.reddit.com/r/rakulang/comments/znkul3/the_rakuju_hijack_hack_of_d3js/).  Check it out!

### Migration your MacOS Photos Folders
*Pawel Pabian* has written a blog post about a very interesting use of Raku: [migrating your pictures out of MacOS Photos Folders](https://dev.to/bbkr/migrate-macos-photos-folders-and-albums-to-plain-tree-of-directories-2c1) into something less Apple-centric.  Yours truly will certainly have a look at that!

### 10 Raku Best Practices

Out of the blue in what appears to be somewhat AI-generated, *Agnes Meyer* provides a list of [Raku's 10 best practices](https://climbtheladder.com/10-raku-best-practices/).

### Raku Advent Calendar 2022

The past weeks entries of the fourth Raku Advent Calendar:

- [Day 13: Virtuel Environments in Raku](https://raku-advent.blog/2022/12/13/virtual-environments-in-raku/) by *Tony-O'Dell*.
- [Day 14: Trove – yet another TAP harness](https://raku-advent.blog/2022/12/14/day-14-trove-yet-another-tap-harness/) by *Mikhael Knarkov* ([/r/rakulang](https://www.reddit.com/r/rakulang/comments/zlnugw/day_14_trove_yet_another_tap_harness_pheix_team/) comments).
- [Day 15: Junction transformers](https://raku-advent.blog/2022/12/15/day-15-junction-transformers/) by *Ben Davies*.
- [Day 16: Santa CL::AWS (part 2)](https://raku-advent.blog/2022/12/16/day-16-santa-claws-part-2/) by *Steve Roe*.
- [Day 17: How to clarify which parts of the documentation change](https://raku-advent.blog/2022/12/17/day-17-how-to-clarify-which-parts-of-the-documentation-change/) by *Richard Hainsworth*.
- [Day 18: Something else](https://raku-advent.blog/2022/12/18/day-18-something-else/) by *Elizabeth Mattijsen*.
- [Day 19: A few modules to ease working with databases in Raku applications](https://raku-advent.blog/2022/12/19/day-19-a-few-modules-to-ease-working-with-databases-in-raku-applications/) by *Jonathan Worthington*.

### FOSDEM 2023

The TPRF has been able to secure a booth at FOSDEM 2023, and is [looking for people to populate the booth](https://news.perlfoundation.org/post/fosdemstand)!

### The SF Raku Study Group

The Raku Study Group will have another [online meeting on New Year's Day](https://www.reddit.com/r/rakulang/comments/zn8aa8/the_sf_perl_raku_study_group/).

### Weeklies

[Weekly Challenge #196](https://theweeklychallenge.org/blog/perl-weekly-challenge-196/) is available for your perusal.

### New Problem Solving Issues

- [Handling of non-breaking spaces when splitting to words](https://github.com/Raku/problem-solving/issues/357)
- [Request to deprecate untwigiled attributes](https://github.com/Raku/problem-solving/issues/358)

### New Pull Requests

- [Consolidate `supply` from `statement-prefixes.pod6` to `control.pod6`](https://github.com/Raku/doc/pull/4173)
- [Consolidate `gather` from `statement-prefixes.pod6` to `control.pod6`](https://github.com/Raku/doc/pull/4174)
- [Speedup creating `sha1` digest string](https://github.com/MoarVM/MoarVM/pull/1731)
- [Fix swapped iterators in ACCEPTS](https://github.com/rakudo/rakudo/pull/5133)
- [Allow `*` to be used as identity on `classify`|`categorize`](https://github.com/rakudo/rakudo/pull/5140)

### Core Developments

- *Florian Weimer* prevented a future issue around implicit function declarations in MoarVM.
- *Daniel Green* JITted various less used nqp ops, such as `nqp::rand_i`.
- *Vadim Belman* implemented the new `use experimental :rakuast` feature, allowing access to the `RakuAST::` clases, and made error reporting from `RakuAST` classes better.
- *Elizabeth Mattijsen* made `List.head()` about 2.4x as fast (now faster than `List[0]`).
- And some other smaller fixes and improvements.

### Questions about Raku

- [How to align strings to right, and chop them if too long?](https://stackoverflow.com/questions/74787524/how-to-align-strings-to-right-and-chop-them-if-too-long-in-raku) by *menfon*.
- []How can you use multiple modules in a Raku project, if the modules are defined in the project?()https://stackoverflow.com/questions/74788277/how-can-you-use-multiple-modules-in-a-raku-project-if-the-modules-are-defined-i by *Rawley Fowler*.
- [`Readline` module support?](https://www.reddit.com/r/rakulang/comments/zm4ek4/readline_module_support/) by *markjreed*.
- [How to slip `gather`-`take` in lazy manner into `map`?](https://stackoverflow.com/questions/74814974/how-to-slip-gather-take-in-lazy-manner-into-map) by *Pawel Pabian*.
- Why is my `%h is List = 1,2;` a valid assignment?[](https://stackoverflow.com/questions/74846224/why-is-my-h-is-list-1-2-a-valid-assignment) by *Daniel Sockwell*.
- [Compiling rakudo on a Raspberry Pi 3B+](https://www.reddit.com/r/rakulang/comments/zprcj0/compiling_rakudo_on_a_raspberry_pi_3b/) by *Marcool04*.

### Meanwhile on Mastodon

- [Grammars and multi-dispatch](https://glasgow.social/@scimon/109506149774594841) by *Simon Proctor*.
- [Recursive with 5 terminators](https://g0v.social/@gugod/109507070054463601) by *Kang-min Liu*.
- [Slightly obsessed with beauty](https://fosstodon.org/@codesections/109507299627574800) by *Daniel Sockwell*.
- [Some font resizing](https://glasgow.social/@scimon/109512113051162578) by *Simon Proctor*.
- [An easier way without Turing machine](https://glasgow.social/@scimon/109518001242614064) by *Simon Proctor*.
- [Confused?  You won't be after...](https://hackers.town/@randomgeek/109524495714596462) by *Brian Wisti*.
- [Experiencing some envy](https://fosstodon.org/@jns/109525155532360399) by *Jonathan Stowe*.
- [A commandline time tracker](https://connectified.com/@masukomi/109525283140601600) by *Kay Rhodes*.
- [What is too long](https://fosstodon.org/@codesections/109529454383403459) by *Daniel Sockwell*.
- [Adding up to something](https://fosstodon.org/@codesections/109535773046613306) by *Daniel Sockwell*.
- [Our sister language](https://social.sdf.org/@mjgardner/109536520998056782) by *Mark Gardner*.
- [PSA for a couPle of annoying bugs](https://social.joelle.us/@joelle/109535997907275706) by *Joelle Maslak*.
- [Disappointed by META](https://social.joelle.us/@joelle/109541521168029384) by *Joelle Maslak*.

### Meanwhile, still on Twitter

- [Updating `Github::Actions`](https://twitter.com/jjmerelo/status/1602242754205057026) by *JJ Merelo*.
- [Innovative dress sense](https://twitter.com/chrisjej/status/1602521077216837632) by *Chris Jack*.
- [Landing in core](https://twitter.com/jjmerelo/status/1602613705597919233) by *JJ Merelo*.
- [Testing in and output](https://twitter.com/jjmerelo/status/1602975717826134017) by *JJ Merelo*.
- [By another name](https://twitter.com/ksnk1040/status/1603600162714456064) by *コシヌケ1040*.
- [Impossible to beat](https://twitter.com/superponime/status/1603673673025564672) by *ポルノアニメ(ITと人権)*.
- [Started working on my contribitions](https://twitter.com/cpan_author/status/1604421264071921664) by *Mohammad S Anwar*.
- [Finding out which native modules are istalled](https://twitter.com/jjmerelo/status/1604580437027946499) by *JJ Merelo*.
- [Simple native libraries](https://twitter.com/jjmerelo/status/1604754483007094784) by *JJ Merelo*.

### Comments

- [Part of the chain](https://github.com/mame/quine-relay#what-this-is) by *Yusuke Endoh*.
- [Unified static and dynamic typing](https://www.reddit.com/r/ProgrammingLanguages/comments/zhs0zd/comment/izzgygy/) by *Ralph Mellor*.
- [YATH - Yet Another Test Harness](https://news.ycombinator.com/item?id=33980679) by *Mickael Knarkhov*.
- [On Rationals vs Floats](https://www.reddit.com/r/ProgrammingLanguages/comments/zju3b0/comment/izzpzsi/) by *Ralph Mellor*.
- [Properly specified](https://news.ycombinator.com/item?id=33988993) by *Grn0ti*.
- [A trait for shallowness](https://www.reddit.com/r/rakulang/comments/zg4rrq/comment/j08z4wc/) by *Ralph Mellor*
- [A great environment](https://news.ycombinator.com/item?id=33997623) by *Grn0ti*.
- [Remarkably bitter](https://news.ycombinator.com/item?id=33999198) by *counterpartyrsk*.
- [Please pick up more steam](https://news.ycombinator.com/item?id=34001015) by *0x445442*.
- [Make making programming languages easier](https://www.reddit.com/r/ProgrammingLanguages/comments/zp3zmv/comment/j0rz8e2/) by *Ralph Mellor*.
- [One day with hope](https://news.ycombinator.com/item?id=34044845) by *cardanome*.

### New Raku Modules

- [Javascript::D3](https://raku.land/zef:antononcube/JavaScript::D3) "Generation of JavaScript's D3 code for making plots and charts" by *Anton Antonov*.
- [Native::FindVersion](https://raku.land/zef:jjmerelo/Native::FindVersion) "Find the last, or only, installed version of a shared lib" by *JJ Merelo*.

### Updated Raku Modules

- [Raku::Pod::Render](https://raku.land/zef:finanalyst/Raku::Pod::Render), [Collection](https://raku.land/zef:finanalyst/Collection) by *Richard Hainsworth*.
- [Trove](https://raku.land/zef:knarkhov/Trove) by *Mikhael Knarkhov*.
- [IO::Capture::Simple](https://raku.land/zef:jjmerelo/IO::Capture::Simple), [cmark::Simple](https://raku.land/zef:jjmerelo/cmark::Simple) by *JJ Merelo*.
- [Jupyter::Kernel](https://raku.land/cpan:BDUGGAN/Jupyter::Kernel) by *Brian Duggan*.
- [Prettier Table](https://raku.land/zef:masukomi/Prettier::Table) by *Kay Rhodes*.
- [Sparky::JobApi](https://raku.land/zef:melezhik/Sparky-Job-Api) by *Alexey Melezhik*.
- [MeCab](https://raku.land/zef:titsuki/MeCab) by *Toyota Itsuki*.
- [Net::BGP](https://raku.land/cpan:JMASLAK/Net::BGP) by *Joelle Maslak*.
- [EC](https://raku.land/zef:grondilu/EC) by *Lucien Grondin*.
- [Intl::CLDR](https://raku.land/zef:guifa/Intl::CLDR) by *Matthew Stuckwisch*.
- [MongoDB](https://raku.land/cpan:MARTIMM/MongoDB) by *Marcel Timmerman*.
- [Red](https://raku.land/zef:FCO/Red) by *Fernando Correa de Oliveira*.

### Winding down

A very cold week that reminded mew how lucky yours truly is, compared to many people in Ukraine who have no heating or water or electricuty, and who are still fighting the Russian aggression.  Слава Україні!  Героям слава!

Please keep staying safe, keep staying healthy, and keep up the good work!

If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal!
