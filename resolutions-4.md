# Resolutions #3

> This is a third follow-up on [Raku Resolutions](https://dev.to/lizmat/series/34948) series.

The third meeting was held on 21 February 2026 at 19:00 UTC.  Apart from 3 Raku Steering Council members, only 1 other person attended.  In the end, **7** issues were discussed within the allotted time (1 hour).

### [Separate Community Resource page](https://github.com/Raku/problem-solving/issues/286)

After some discussion, it became clear that this issue basically has gone stale in light of the new raku.org site.  *Richard Hainsworth* will create a new issue, after which this one can be closed.

### [What makes `unit` be too late?](https://github.com/Raku/problem-solving/issues/501)

After some discussion it was agreed that from a language design point of view, it shouldn't really matter when the `unit` occurs in the code, as long as there's only one of them.  And that the rest of the source would be considered to be part of that scope.

It was agreed that the documentation will describe this future state with the caveat that this is currently not yet the case.  *Eric Forste* agreed to make a PR accordingly, after which the issue can be closed.

The point of being able to specify an `EXPORT` sub *before* a `unit` would be technically correct, but not very useful in practice as an `EXPORT` sub usually needs to refer to objects/classes that have already been defined in the code (and thus should logically be positioned *after* the scope).

### [Errors indexing past the end of a `List`](https://github.com/Raku/problem-solving/issues/160)

After some discussion it was decided that the current behaviour of returning `Nil` for `List` elements beyond the end is correct.  And that returning a `Failure` for negative index values is the most consistent behaviour, especially in light of `Array` doing that as well.  So the issue could be closed.

### [Some useful math/statistics functions are missing](https://github.com/Raku/problem-solving/issues/4)

Consensus was that it would be nice to have all of these math / statistics methods, but that these should live in module land for the foreseeable future.  So the issue could be closed.

### [There's a huge PR/issue deficit in the Rakudo repo](https://github.com/Raku/problem-solving/issues/5)

It was recognized that there are indeed quite a few open issues in Rakudo (although much less than there have been in the past).  However, there will always be open issues.  And with the current size of the core team, the number of issues will not significantly reduce in the future (with the current rate of new issues coming in).

So it was decided to close the issue as "unresolvable".

### [New named parameters to `.classify`](https://github.com/Raku/problem-solving/issues/6)

It was decided that the issue has gone a bit stale, and thus ask the OP whether this should stay open, especially since it was suggested that `.classify` / `.categorize` maybe should need an overhaul (at least internally).

### [Need a substitute for Perl 5 `die` with newline for raising end-user errors?](https://github.com/Raku/problem-solving/issues/59)

After explaining the esoteric rule in Perl 5 with regards to `die` with a message with and without a new line, it was agreed that it would be nice to have such a feature.  Especially since at least 2 of the attendees had been using a `note $message; exit 1` sequence to achieve just that.  Disadvantage to this is that such a sequence can **not** be caught by a `CATCH`.

After some discussion, a `:no-backtrace` named argument to `die()` was suggested, and that the issue could be closed if a Pull Requst had been created for this.  This has since then been implemented in [#6076](https://github.com/rakudo/rakudo/pull/6076), so the issue was closed.

## Next meeting

The next meeting will be held at 7 March 2026 at 19:00 UTC (20:00 CET, 14:00 EST, 11:00 PST, 04:00 JST (22 Jan), 06:00 AEST (22 Jan)), and again at a one hour maximum. If not all of these issues have been resolved, they will be moved to a future meeting.

Since Jitsi is still working out so far, the next one will be held at the same URL: [https://meet.jit.si/SpecificRosesEstablishAllegedly](https://meet.jit.si/SpecificRosesEstablishAllegedly). The reason Jitsi was selected, is that it has proven to be working with minimal hassle for at least the Raku Steering Council meetings. As the only thing you need to be able to attend, is a modern browser, a camera, and a microphone. No further installation required.

A new [set of issues](https://github.com/Raku/problem-solving/issues?q=state%3Aopen%20label%3A%22Next%20Resolutions%20Meeting%22) has been selected by yours truly (in issue number order, oldest first):
- [Specify rounding mode in CORE](https://github.com/Raku/problem-solving/issues/13)
- [Metadata licenses should be required before adding new modules to ecosystem](https://github.com/Raku/problem-solving/issues/19)
- [Semantics of coercion type on an `rw` parameter](https://github.com/Raku/problem-solving/issues/21)
- [where blocks vs sub signatures](https://github.com/Raku/problem-solving/issues/24)
- [LCM (support `Rat` and fail on lossy coercion)](https://github.com/Raku/problem-solving/issues/55)
- [Should `without` allow chaining?](https://github.com/Raku/problem-solving/issues/63)
- [`.raku` should be replaced with a pluggable system](https://github.com/Raku/problem-solving/issues/91)

## Preparation

Any Raku Community member is welcome to these meetings.  Do you consider yourself a Raku Community member?  You're welcome.  It's as simple as that.

But please make sure you have looked at the issues that will be discussed **before** attending the meeting.  And if you already have any comments to make on these issues, make them with the issue beforehand.

The original contributors to these issues will also be notified (unless they muted themselves from these issues).  We hope that they also will be able to attend.

## Small steps

If you consider yourself a Raku Community member, please try to attend!  If anything, it will allow you to put faces to the names that you may be familiar with.

Hope to see you there!
