# Day 25 – Raku 2024 Review

2024 was a year of changes and firsts.

## Edument

Edument Central Europe, the branch of Edument that is based in Prague (and led by *Jonathan Worthington*), decided to stop (commercial) development of its Raku related products: [`Comma`](https://commaide.com) (the IDE for the Raku Programming Language) and [`Cro`](https://cro.services) (a set of libraries for building reactive distributed systems).

### Comma

The [announcement](https://commaide.com/discontinued):

> With the discrepancy between revenue and development cost to continue being so large, and the prevailing economic environment forcing us to focus on business activities that at least pay for themselves, we’ve made the sad decision to discontinue development of the Comma IDE.

Fortunately, *Jonathan Worthington* was able to release all of the sources of Comma, so that other people could continue Comma development.  With *John Haltiwanger* being the now de-facto project leader.  This has resulted in a beta of the open source version of a Raku plugin for IntelliJ IDEA, as described in an [advent post]( https://raku-advent.blog/2024/12/20/day-20-re-introducing-a-raku-plugin-for-intellij-idea/)

### Cro

The [announcement]( https://web.archive.org/web/20240421151537/https://cro.services/community-transfer):

> When Edument employees were by far the dominant Cro contributors, it made sense for us to carry the overall project leadership. However, the current situation is that members of the Raku community contribute more. We don’t see this balance changing in the near future.

> With that in mind, we entered into discussions with the Raku Steering Council, in order that we can smoothly transfer control of Cro and its related projects to the Raku community. In the coming weeks, we will transfer the GitHub organization and release permissions to steering council representatives, and will work with the Raku community infrastructure team with regards to the project website.

As the source code of `Cro` had always been open source, this was more a question of handing over responsibilities.  Fortunately the Raku Community reacted: *Patrick Böker* has taken care of making `Cro` a true open source project related to Raku, and the [associated web site https://cro.raku.org](https://cro.raku.org) is now being hosted on the Raku infrasructure.  With many kudos to the Raku Infra Team!

### A huge thanks

Sadly, *Jonathan Worthington* also indicated that they would only remain minimally involved in the further development of [MoarVM](https://github.com/MoarVM/MoarVM), [NQP](https://github.com/raku/NQP) and [Rakudo](https://github.com/rakudo/rakudo/) in the foreseeable future.  As such, all of their modules were moved to the [Raku Community Modules Adoption Center](https://github.com/raku-community-modules), where they were updated and re-released.



## Issues cleanup

At the beginning of October, *Elizabeth Mattijsen* decided to take on the large number of open Rakudo issues at that time: 1300+.  They reported on that work in [Raku Fall Issue Cleanup](https://dev.to/lizmat/raku-fall-issue-cleanup-lkc), which resulted in the closing of more than 500 issues.  Of the about 800 open issues remaining, almost 300 were marked as "fixed in RakuAST", and abou 100 were marked as "Will be addressed in RakuAST".  Which still leaves about 400 open, so there's still plenty of work to be done here.

## RakuDoc v2.0

Rakudo saw about 2000 commits (MoarVM, NQP, Rakudo, doc) this year, which is about the same as in 2023.  About one third of these commits were in the development of RakuAST (down from 75% in 2023).

Commits: rakudo: 1390, MoarVM: 145, NQP: 289, docs: 289

p6c:
- 658 -> 427 modules
- 230 -> 138 unique authors

Ecosystem:
332 -> 557 modules 1st release / update  2023 -> 2024

RakuAST:
make test: 110/151 (73%) -> 140 / 156 (90%)
make spectest: 980/1356 (72%) -> 1155 / 1359 (85%)

Rakudo:
- dispatcher logic simplified by introducing nqp::shortcuts
- MOP in Rakudo / NQP streamlined
- add "is item" dispatch disambiguation trait
- sub form of "trans" + complete overhaul (5% -> 3x as fast)
- add "is-revision-gated" trait
- additions: Parameter.of, Allomorph.narrow, Any.are(type),
  Parameter.new(:invocant), .flat(:hammer), .min/.max/.minmax(:by)
  .roles on enums, VM.remote-debugging
- additions 6.e: Cool.nomark, Date.DateTime timezone aware,
  .contains/.starts-with/.ends-with/.index/.rindex/.substr-eq
  now have :smartcase, IO::Path.stem, any junction regex
  interpolation, vulgar fractions in val()
- CI testing usable again (no more false positives)
- Up to 2x as fast on JVM backend
- disabled Expression JIT by default (5% faster on Intel)
- add @a[**], %h{**} for :hammer ing behaviour

RakuAST:
- RakuDoc v2.0
- syntax highlighting of compilable source code

Important fixes:
- race condition in lazy deserialization on MoarVM backend
- allow enums in MAIN processing that shadow core types
- make sure LAST phaser in .map always fires
- no unnecessary Failures on numeric ops
- Make "AAS" .. "ABS" use standard .succ semantics
- stop infiniloops on stringification of self-referencing
  (Quant)Hashes
- Failures in slices will always throw, instead of silently
  shortcutting lazy slices
