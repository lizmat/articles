# The first four

> This is a first follow-up on [Raku Resolutions](https://dev.to/lizmat/raku-resolutions-17g7)

The first meeting was had on the suggested time and date: 17 January 2026 at 19:00 UTC.  Apart from 4 Raku Steering Council members, up to 8 other people attended: thank you for your attendance and your feedback.

In the end, **4** issues were discussed within the allotted time (1 hour) and one other was explicitely moved to a next meeting at the request of the issuer.

## [Specify documentation URLs](https://github.com/Raku/problem-solving/issues/93)

*Richard Hainsworth* explained some of the background of the issue.  At one point in the creation of the Raku documentation, it was decided that the documentation of methods with the same name should be grouped in pages automatically generated from the original source documentation (for instance, documentation of the [`print` method](https://docs.raku.org/routine/print)).

Over the years, these pre-generated pages got incoming links from other parts of the Raku documentation.  But the URLs of the pre-generated pages could shift (and have) over time, thus making the links dead.

In the new Raku Documentation rendering (based on RakuDoc) this issue will need / is being handled in a more programmatic manner, but this will also require some cleanup.

It was therefore decided to close this issue for now, with an [explanation as to why](https://github.com/Raku/problem-solving/issues/93#issuecomment-3764321909) and an option to create a new *documentation* issue once that makes sense.

## [Generic name for `FALLBACK` and `CALL-ME`](https://github.com/Raku/problem-solving/issues/119)

A lot of discussion was had about this issue, but it was clear that a generic name, especially "hook", would **not** be a good idea from a documentation point of view.

It also became clear that nobody present had complete knowledge about all of the nooks and crannies of these special cases in the Raku Programming Language.

*Elizabeth Mattijsen* suggested that someone should really write a series of blog posts about these special features, and start categorizing them by when they are being called, and what function they perform.  And whether they can be customised / overridden by user code or are indeed needed to be specified in user code to be functioning.  And that that series of blog posts could then serve as a base for extended documentation about these features.

And that the issue could be closed with a [mention](https://github.com/Raku/problem-solving/issues/119#issuecomment-3765238401).

## [SEO for the keyword "rakulang" seems poor](https://github.com/Raku/problem-solving/issues/147)

Everybody agreed that this has become a non-issue in the 6 years after this issue was originally posted.  And that it could be closed with a [mention](https://github.com/Raku/problem-solving/issues/147#issuecomment-3765176471).

## [`Seq` vs. `List` - `Iterable`, instead?](https://github.com/Raku/problem-solving/issues/499)

This had a lot of discussion.  It became clear that this both has documentation, as well as spec testing (roast) fallout.  Since changes to roast indicate language changes, it will probably require a language level bump for the new return types to be actually more leniently tested.

For example, code in the ecosystem might depend on e.g. a mutable `Array` being returned by a core method: changing it to a more lenient `Iterable` return type would allow the implementation to return a more memory efficient immutable `List` instead.  By putting this change in a language level would allow the code in the  ecosystem to opt out of this, by requiring the language level (where that method was) guaranteed to return an `Array`.

So it was decided that the documentation would start using `Iterable`, but the actual changes in the spec tests (roast) would need to be guarded by a language level bump.  And so it was [noted](https://github.com/Raku/problem-solving/issues/499#issuecomment-3764294232).

## The next meeting

After the hour was done, it was felt it was a good idea to continue this type of meeting.  And it was said that we could meet again in two weeks.

However, in two weeks [FOSDEM](https://fosdem.org/2026) will be in full swing, and at least two of the Raku Steering Council members won't be able to attend because of it (more specifically, because of the [Community Dinner](https://perlfoundation.org/fosdem/community-dinner.html)).

So the next meeting will be held at 7 February 2026 at 19:00 UTC (20:00 CET, 14:00 EST, 11:00 PST, 04:00 JST (18 Jan)), and again at a one hour maximum.  If not all of these issues have been resolved, they will be moved to a future meeting.

Since Jitsi worked out ok for this meeting, the next one will be held at the same URL: [https://meet.jit.si/SpecificRosesEstablishAllegedly](https://meet.jit.si/SpecificRosesEstablishAllegedly).  The reason Jitsi was selected, is that it has proven to be working with minimal hassle for at least the Raku Steering Council meetings.  As the only thing you need to be able to attend, is a modern browser, a camera, and a microphone.  No further installation required.

The remaining issues that will be discussed, are:

- [Separate Community Resource pages](https://github.com/Raku/problem-solving/issues/286)
- [Function return types should also tell about the used assignment/container](https://github.com/Raku/problem-solving/issues/337)
- [Raku Classification System](https://github.com/Raku/problem-solving/issues/489)
- [What makes `unit` be too late?](https://github.com/Raku/problem-solving/issues/501)

## Preparation

Any Raku Community member is welcome to this meeting.  Do you consider yourself a Raku Community member?  You're welcome.  It's as simple as that.

But please make sure you have looked at the issues that will be discussed **before** attending the meeting.  And if you already have any comments to make on these issues, make them with the issue beforehand.

The original contributors to these issues will also be notified (unless they muted themselves from these issues).  We hope that they also will be able to attend.

## Small steps

If you consider yourself a Raku Community member, please try to attend!  If anything, it will allow you to put faces to the names that you may be familiar with.

Hope to see you there!
