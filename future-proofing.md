# Future-proofing the Raku Programming Language

Around this time of year, *Jonathan Worthington* was writing their Advent Post called [Reminiscence, refinement, revolution](https://raku-advent.blog/2020/12/25/day-25-reminiscence-refinement-revolution/).  Today, yours truly finds themselves writing a similar blog post after what can only be called a peculiar year in the world.

## Visible highlights

The Raku Programming Language, as a language, has not seen lot of development this year, as most of the work has been "under the hood".  The most visible highlights in the Raku Programming Language are basically:

### last / next with value

````raku
use v6.e.PREVIEW;
say (^5).grep: { $_ == 3 ?? last False !! True } # (0 1 2)
say (^5).grep: { $_ == 3 ?? last True  !! True } # (0 1 2 3)
````
Normally, `last` just stops an iteration, but now you can give it a value as well, which can be handy e.g. in a `grep` where you know the current value is the last, but you still want to include it.
````raku
use v6.e.PREVIEW;
say (^5).map: { next    if $_ == 2; $_ } # (0 1 3 4)
say (^5).map: { next 42 if $_ == 2; $_ } # (0 1 42 3 4)
````
Similarly with `map`, if you want to skip a value (which was already possible), you can now replace that value by another value.

Note that you need to activate the upcoming `6.e` Raku language level to enable this feature, as there were some potential issues when activated in `6.d`.  But that's just one example of future proofing the Raku Programming Language.

### .pick(\*\*)

The [`.pick(*)`](https://docs.raku.org/routine/pick) call will produce all possible values of the [`Iterable`](https://docs.raku.org/type/Iterable) on which it is called in random order, and then stop.  The `.pick(**)` will do the same, but then start again producing values in (another) random order until exhausted, ad infinitum.
````raku
.say for (^5).pick(* ); # 3␤4␤0␤2␤1␤
.say for (^5).pick(**); # 3␤4␤0␤2␤1␤0␤2␤1␤4␤3␤3␤4␤2␤1␤0␤....
````
Nothing essential, but it is sure nice to have 😀.

### `is implementation-detail` trait

The [`is implementation-detail`](https://docs.raku.org/routine/is-implementation-detail) trait indicates that something that *is* publicly visible, still should be considered off-limits as it is a detail of the implementation of something (whether that is your own code, or the core).  This will also mark that something as invisible for standard introspection:
````raku
class A {
    method foo() is implementation-detail { }
    method bar() { }
}
.name.say for A.^methods; # bar␤BUILDALL␤
````
Subroutines and methods in the core that are considered to be an implementation-detail, have been marked as such.  This should make it more clear which parts of the Rakudo implementation are game, and which parts are off-limits for developers (knowing that they can be changed without notice).  Yet another way to make sure that any Raku programs will continue to work with future versions of the Raku Programming Language.

## Invisible highlights

There were many smaller and bigger fixes and improvements "under the hood" of the Raku Programming Language.  Some code refactoring that e.g. made [`Allomorph`](https://docs.raku.org/type/Allomorph) a proper class, without changing any functionality of [allomorphs](https://docs.raku.org/language/glossary#index-entry-Allomorph) in general.  Or speeding up by using smarter algorithms, or by refactoring so that common hot code paths become smaller than the inlinining limit, and this become a lot faster.

But the **BIG** thing in the past year, was that the so-called ["new-disp" work was merged](https://6guts.wordpress.com/2021/09/29/the-new-moarvm-dispatch-mechanism-is-here/).  In short, you could compare this to ripping out a gasoline engine from a car (with all its optimizations for fuel efficiency of 100+ years of experience) and replacing this by an electrical engine, while its being driven running errands.  And although the electrical engine is much more efficient, it still can gain a lot from more optimizations, which have been applied since the merge.

For yours truly, the notion that it is better to remove certain optimizations written in `C` in the virtual engine, and replace them by code written in `NQP`, was the most eye-opening one.  The reason for this is that all of the optimization work that `MoarVM` does, can only work on the parts it understands at runtime.  And `C` code, is not what `MoarVM` understands, so it can not optimize that at runtime.

The current state of this work, is that it for now is a step forward, but also a step back in some aspects (at least for now).  Some workflows most definitely have benefitted from the work so far (especially if you dispatch on anything that has a `where` clause in it, or use [`NativeCall`](https://docs.raku.org/language/nativecall) directly, or indirectly with e.g. [`Inline::Perl5`](https://raku.land/cpan:NINE/Inline::Perl5#description)).  But the bare startup time of Rakudo has increased.  Which has its effects on the speed with modules are installed, or testing is being done.

The really good thing about this work, is that it will allow more people to work on optimizing Rakudo, as that optimizing work can now be done in `NQP`, rather than in `C`.  The next year will most definitely see one or more blog posts and/or presentations about this, to lower the already lowered threshold even further.

## The Ecosystem

Ecosystem / Raku Land

Preserving history

CCR

IRC Logs


