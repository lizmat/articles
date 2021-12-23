# Future-proofing the Raku Programming Language

Around this time of year, *Jonathan Worthington* was writing their Advent Post called [Reminiscence, refinement, revolution](https://raku-advent.blog/2020/12/25/day-25-reminiscence-refinement-revolution/).  Today, yours truly finds themselves writing a similar blog post after what can only be called a peculiar year in the world.

## The Language

The Raku Programming Language, as a language, has not seen lot of development this year, as most of the work has been "under the hood".

### Visible highlights

The most visible highlights in the Raku Programming Language are basically:

#### `last` / `next` with a value

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

#### `.pick(**)`

The [`.pick(*)`](https://docs.raku.org/routine/pick) call will produce all possible values of the [`Iterable`](https://docs.raku.org/type/Iterable) on which it is called in random order, and then stop.  The `.pick(**)` will do the same, but then start again producing values in (another) random order until exhausted, ad infinitum.
````raku
.say for (^5).pick(* ); # 3␤4␤0␤2␤1␤
.say for (^5).pick(**); # 3␤4␤0␤2␤1␤0␤2␤1␤4␤3␤3␤4␤2␤1␤0␤....
````
Nothing essential, but it is sure nice to have 😀.

#### `is implementation-detail` trait

The [`is implementation-detail`](https://docs.raku.org/routine/is-implementation-detail) trait indicates that something that *is* publicly visible, still should be considered off-limits as it is a detail of the implementation of something (whether that is your own code, or the core).  This will also mark that something as invisible for standard introspection:
````raku
class A {
    method foo() is implementation-detail { }
    method bar() { }
}
.name.say for A.^methods; # bar␤BUILDALL␤
````
Subroutines and methods in the core that are considered to be an implementation-detail, have been marked as such.  This should make it more clear which parts of the Rakudo implementation are game, and which parts are off-limits for developers (knowing that they can be changed without notice).  Yet another way to make sure that any Raku programs will continue to work with future versions of the Raku Programming Language.

### Invisible highlights

There were many smaller and bigger fixes and improvements "under the hood" of the Raku Programming Language.  Some code refactoring that e.g. made [`Allomorph`](https://docs.raku.org/type/Allomorph) a proper class, without changing any functionality of [allomorphs](https://docs.raku.org/language/glossary#index-entry-Allomorph) in general.  Or speeding up by using smarter algorithms, or by refactoring so that common hot code paths become smaller than the inlinining limit, and this become a lot faster.

But the **BIG** thing in the past year, was that the so-called ["new-disp" work was merged](https://6guts.wordpress.com/2021/09/29/the-new-moarvm-dispatch-mechanism-is-here/).  In short, you could compare this to ripping out a gasoline engine from a car (with all its optimizations for fuel efficiency of 100+ years of experience) and replacing this by an electrical engine, while its being driven running errands.  And although the electrical engine is much more efficient, it still can gain a lot from more optimizations.

For yours truly, the notion that it is better to remove certain optimizations written in `C` in the virtual engine, and replace them by code written in `NQP`, was the most eye-opening one.  The reason for this is that all of the optimization work that `MoarVM` does at runtime, can only work on the parts it understands.  And `C` code, is not what `MoarVM` understands, so it can not optimize that at runtime.  Simple things such as assignment, had been optimized in `C` code, and basically had become an "island" of unoptimization.  But no more!

The current state of this work, is that it for now is a step forward, but also a step back in some aspects (at least for now).  Some workflows most definitely have benefitted from the work so far (especially if you dispatch on anything that has a `where` clause in it, or use [`NativeCall`](https://docs.raku.org/language/nativecall) directly, or indirectly with e.g. [`Inline::Perl5`](https://raku.land/cpan:NINE/Inline::Perl5#description)).  But the bare startup time of Rakudo has increased.  Which has its effects on the speed with modules are installed, or testing is being done.

The really good thing about this work, is that it will allow more people to work on optimizing Rakudo, as that optimizing work can now be done in `NQP`, rather than in `C`.  The next year will most definitely see one or more blog posts and/or presentations about this, to lower the already lowered threshold even further.

In any case, kudos to *Jonathan Worthington*, *Stefan Seifert*, *Daniel Green*, *Nicholas Clark* and many, many others for pulling this off!

## The Ecosystem

Thanks to [`Cro`](https://cro.services), a set of libraries for building reactive distributed systems (lovingly crafted to take advantage of all Raku has to offer), a number of ecosystem related services have come into development and production.

### zef ecosystem

The new `zef` ecosystem has become of age and is now support by various developer apps, such as [`App::Mi6`](https://raku.land/cpan:SKAJI/App::Mi6#synopsis), which basically reduces the distribution upload / commit process to a single `mi6 release↵`.  Recommended by yours truly, especially if you are about to develop a Raku module from scratch.  There are a number of advantages to using the `zef` ecosystem:

#### direct availability

Whenever you upload a new distribution to the `zef` ecosystem, it becomes (almost) immediately available for download by users.  This is particularly handy for CI situations, if you are first updating one or more dependencies of a distribution: the `zef` CLI wouldn't know about your upload upto an hour later.

#### better ecosystem security

Distributions from the older ecosystem backends could be removed by the author without the ecosystem noticing it (p6c), or not immediately noticing it (CPAN).  Distributions, once uploaded to the `zef` ecosystem, can not be removed.

#### more dogfooding

The `zef` ecosystem is completely written in the Raku Programming Language itself.  And you could argue that's one more place where Raku is in production.  Kudos to *Nick Logan* and *Tony O'Dell* for making this all happen!

### raku.land

The [raku.land](https://raku.land) is a website entirely developed with the Raku Programming Language as a place to browse the Raku ecosystem.  It should be seen as the successor of the [modules.raku.org](https://modules.raku.org) website, which is **not** based on Raku itself.  Although some of the features are still missing, it is an excellent piece of work by *James Raspass* and very much under continuous development.



Ecosystem / Raku Land

Preserving history

CCR

IRC Logs


