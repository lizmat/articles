# An end to magical madness

While going through all currently open Rakudo issues (as described in [Raku Fall Issue Cleanup](https://dev.to/lizmat/raku-fall-issue-cleanup-lkc), I was reminded of a project that I started in April 2020 to make the semantics of the [`...` aka "sequence operator"](https://docs.raku.org/routine/%2E%2E%2E) more sane and more performant.  And that I came to the [conclusion](https://irclogs.raku.org/raku-dev/2020-04-19.html#12:37) that continuing that project (within the confines of Rakudo) would be a losing battle.

So I created [a repository](https://github.com/lizmat/Sequence-Generator), worked on that for about two weeks.  And then stopped.  And then forgot all about it until a few days ago.

I have no idea why I stopped.  It was early in the Covid pandemic, so maybe I just got depressed.  And disheartened by the immense number of [still failing tests](https://github.com/lizmat/Sequence-Generator/blob/main/t/02-exhaustive.rakutest#L335-L457).

## Reacquainting

It was weird seeing quite a lot of code that I had written then (more than 4.5 years ago), that looked familiar, but of which I had no active recollection.  The code was pre new-dispatch.  And pre zef-ecosystem.

It definitely had my style of coding, generally.  Still it was oddly unfamiliar in some aspects.  Weird how one's coding style changes over the years.  More experience?  Learned better ways of building class hierarchies?  Possibly.

In any case, in order to familiarize myself with the code again, and whether it would be worthwhile to bring that project to fruition, I decided to bring the code up to my own current style / "standards".  With about 1200 lines of code, and much of it NQP, that initially felt a bit daunting.  But once done, it felt familiar again and some of the concepts that I had been using felt again as the right move forward.

## First release

After having done all that, and actually fixing some of the test failures, it was time to look at the still unsolved Rakudo issues that were [marked with "... magical madness"](https://github.com/rakudo/rakudo/labels/...%20magical%20madness).  The first issue I looked at was about the [elucidation of sequences with complex numbers](https://github.com/rakudo/rakudo/issues/3344).  That was actually actually an easy fix: it was almost more work to add a test for it.

With that done, it was time to release the module to the Raku ecosystem for the first time: [`Sequence::Generator`](https://raku.land/zef:lizmat/Sequence::Generator).  Which gave me the opportunity to comment on an open Rakudo issue, stating it would be fixed if they'd `zef install` and `use` this module.

There still a lot of work to be done on the module though.  But at least it's now usable and if your case of using the `...` operator is covered by this module, then you'd have 4x to 20x times faster support for it.

And if you've found a bug, or missed some functionality, there's now a dedicated [place to let your grievances be known](https://github.com/lizmat/Sequence-Generator/issues/new).  And a good chance they will be acted upon more quickly.

## A benchmark
So how much faster is the module really?

Let's look at a simple benchmark producing all of the odd numbers below 1000.  Because imports in the Raku Programming Language are always lexical, it is possible to test both versions of the infix `...` operator in a single program:
```raku
# number of time to run code
my int $times = 10000;

# simple timer logic returning number of nanoseconds per iteration
sub timer(&code) {
    my $then = now;
    (^$times).map: &code;
    now - $then
}

# using the core infix ... logic
my $old = timer {
    my @a = 1,3,5 ... 1000;
}

# using the Sequence::Generator infix ... logic
my $new = timer {
    use Sequence::Generator;
    my @a = 1,3,5 ... 1000;
}

# the null loop
my $nil = timer { Nil }

# the result
printf "%.2fx as fast\n", ($old - $nil) / ($new - $nil);
```
shows up on my computer as:
```
5.37x as fast
```
Of course, your mileage may vary.  But with this code you should be able to do your own `...` benchmarks more easily.

## Conclusion
It was good to recoup the investment I had done in that part of the Rakudo code in 2020.  And hopefully set more steps towards integrating this functionality and these semantics into a future language version of the Raku Programming Language.

If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!
