# Fall Issue Cleanup

About 3 weeks ago I thought it was time to go through the outstanding [Rakudo compiler](https://rakudo.org) issues (an implementation of the [Raku Programming Language](https://raku.org)) to see how many of them would have been fixed by now with the new Raku grammar.

Why? Because we have reached 85+% of test coverage of the new Raku Grammar in [roast, the official Raku test-suite](https://github.com/raku/roast?tab=readme-ov-file#the-official-raku-test-suite).

> Success as defined by the number of test files that *completely* pass without any test failures.  Most othertest files also have tests passing, but they're not 100% clean.

At that point there were 1312 open Rakudo issues, with the oldest being from 2017.

There are now 778 issues left open.  So you could say that's 534 issues closed.  Actually, it is a few more as during this 3 week period, a few new issues were posted.

In this blog post I'll be describing the types of issues that I've seen, and how I handled them.

## JVM backend specific

Sadly, there's not a lot I could do at those specific issues, as my JVM foo is close to zero.  There are currently [22 open issues for the JVM-backend](https://github.com/rakudo/rakudo/issues?q=is%3Aopen+is%3Aissue+label%3AJVM).  If you have any JVM chops, it would be greatly appreciated if you would apply them to solving these Rakudo issues!

## Simply fixed

When I started, the oldest [open issue in the Rakudo repository](https://github.com/rakudo/rakudo/issues?q=is%3Aopen+is%3Aissue) was about 7.5 years old, and effectively lost in the mist of time.  In any case they predated significant changes, such as the new dispatch mechanism.  Not to mention the Raku Programming Language still had a different name then.

> If you're really interested in the mists of time, you could check out the [old issues](https://github.com/Raku/old-issue-tracker/issues?q=is%3Aissue+is%3Aopen) that were migrated from the venerable RT system, with the oldest open issue from 2010!

So going from the oldest issue to the newest, it was a matter of checking if the problem still existed.  And if it appeared fixed, and if it was possible to write a test for it, write a test for it and close the issue.  And sometimes there even was a PR for a test already, so that was even more a no-brainer.

## Marked as fixed, but also marked as "tests needed"

Quite a few number of issues were actually marked as fixed, but were also marked as needing tests.  Sadly, the original author of the issue had not done that or didn't do that after the issue was fixed.  In most cases it was just a few minutes to write a test, test it and commit it.

## Fixed because of the new dispatch logic

After 18 months of work in 2020 and 2021, the [new dispatch mechanism](https://6guts.wordpress.com/2021/09/29/the-new-moarvm-dispatch-mechanism-is-here/) became default in the 2021.10 release of the Rakudo compiler.  Most of the multi method dispatch related issues that were made *before* that time, appeared fixed.  So it just was a matter of writing the tests (if there weren't any yet) and commit them.

## Fixed in RakuAST

While testing all of these issues, I always also tested whether they were fixed in the new Raku grammar, based on the [RakuAST project](https://dev.to/lizmat/rakuast-for-early-adopters-576n) (which is why I started doing this bug hunting streak in the first place).

> Running code with the new Raku grammar, is as easy as prefixing `RAKUDO_RAKUAST=1` to your call to `raku`.  For instance `raku -e '{ FIRST say "first" }'` does not output anything with the legacy grammar.  But with `RAKUDO_RAKUAST=1 raku -e '{ FIRST say "first" }'` it will say "first" because the `FIRST` phaser fires for any block when it is first executed with the new Raku grammar.

And to my surprise, in many cases they were!  For those cases a [special test file](https://github.com/rakudo/rakudo/blob/main/t/12-rakuast/xx-fixed-in-rakuast.rakutest) is being reserved to which tests are added for issues that have been fixed with the new Raku grammar in RakuAST.

These issues are then marked as being [fixed in RakUAST](https://github.com/rakudo/rakudo/issues?q=is%3Aopen+is%3Aissue+label%3A%22fixed+in+RakuAST%22) but left open, so that people are hopefully prevented from creating duplicate issues for problems that apparently haven't been fixed yet.

> A large percentage of these issues appear fixed because they were essentially static optimizer issues, and the new Raku grammar doesn't have any of these compile-time optimisations yet.  So it's important to be able to check for regressions of this type once optimizations are being added back in.  In turn, these static optimizer issues were often caused by the optimizer not having enough or the correct information for doing optimizations.  Which in turn was one of the reasons to start with RakuAST to begin with.

## Not fixed

And then there were the issues that were simply still reporting an existing problem.  Some of them, with the knowledge that I acquired over the years, looked like easy to fix.  So I put in some effort to fix them.  A non-exhaustive list:

- allow `$:F` as placeholder variable with [`use isms`](https://docs.raku.org/language/pragmas#isms)
- reduce number of [`Failure`](https://docs.raku.org/type/Failure) objects when numerically comparing non-numerical values
- re-introduce support for [`+permutations(30)`](https://docs.raku.org/type/List#routine_permutations)
- allow for [`@*ARGS`](https://docs.raku.org/language/variables#@*ARGS) to contain other [`Cool`](https://docs.raku.org/type/Cool) values (apart from strings)
- fix numeric comparators on [`Rat`s](https://docs.raku.org/type/Rat) with `0` denominator (e.g. `<1/0> <=> <-1/0>`)
- fix [magic increment/decrement](https://docs.raku.org/type/Str#method_succ) on `⁰¹²³⁴⁵⁶⁷⁸⁹` superscript characters
- make `--repl-mode=interactive` CLI argument always force an interactive REPL
- add support for [`any`](https://docs.raku.org/type/Junction) junctions in regex interpolation
- add support for Unicode vulgar fractions to [`val()`](https://docs.raku.org/routine/val) (such as `⅓`)
- seamlessly use `rlrwap` as line editor in REPL if no other modules installed
- improvements on several error messages

About 50 of the outstanding issues look like they should be fixable without turning into large projects, so I will be looking at these in the coming days / weeks.

## Feature requests

Some of the open issues were basically feature requests.  Sometimes I felt that they could be easily implemented (such as several error message improvements) so I implemented them.  Others I created a Pull Request for.  And for still others I felt a [problem-solving issue](https://github.com/raku/problem-solving/issues) would be needed (which I then created).  And some I closed, knowing almost 100% they would never be accepted.

> If this was one of *your* issues, and you still feel that feature should become part of the Raku Programming Language, please don't be discouraged!  Looking at a 60+ issues for 3 weeks in a row sometimes made me a bit grumpy at the end of the day.  Please make a [new problem solving issue](https://github.com/Raku/problem-solving/issues/new/choose) in that case!

## Addressed in RakuAST

Many issues looked like they would be more easily solvable in the new Raku grammar with RakuAST.  There are now [289 of them](https://github.com/rakudo/rakudo/issues?q=is%3Aopen+is%3Aissue+label%3A%22addressed+in+RakuAST%22).  These will be next on my list.

## Conclusion

It was an interesting ride through memory lane the past weeks.  With about 200 commits, that's not bad at all!

Note that with these numbers of issues, if I had an error rate of only 1%, there are at least 5 issues that were closed when they shouldn't have been closed.  If you feel that an issue has been closed incorrectly, please leave a comment and I'll re-open them if you cannot do that yourself.

Sadly, because of the additional tests that I wrote, the number of roast test-files passing has now dropped again below the 85% mark.  Still, I do think this is progress, as the errors that they check for would have been encountered during the development of the Raku grammar sooner or later anyway.

Anyway, it was fun being able to close as many issues as I did!  Want to join in the fun?  There are still [778 issues](https://github.com/rakudo/rakudo/issues?q=is%3Aopen+is%3Aissue) open waiting for someone to close them!
