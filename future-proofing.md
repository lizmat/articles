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
use v6.*;
say (^5).map: { next    if $_ == 2; $_ } # (0 1 3 4)
say (^5).map: { next 42 if $_ == 2; $_ } # (0 1 42 3 4)
````
Similarly with `map`, if you want to skip a value (which was already possible), you can now replace that value by another value.

Note that you need to activate the upcoming `6.e` Raku language level to enable this feature, as there were some potential issues when activated in `6.d`.  But that's just one example of future proofing the Raku Programming Language.

### .pick(\*\*)

The `.pick(*)` call will produce all possible values of the `Iterable` on which it is called in random order, and then stop.  The `.pick(**)` will do the same, but then start again producing values in (another) random order until exhausted, ad infinitum.
````raku
.say for (^5).pick(* ); # 3␤4␤0␤2␤1␤
.say for (^5).pick(**); # 3␤4␤0␤2␤1␤0␤2␤1␤4␤3␤3␤4␤2␤1␤0␤....
````
Nothing essential, but it is sure nice to have 😀.

### is implementation-detail trait

The `is implementation-detail` trait indicates that something that *is* publicly visible, still should be considered off-limits as it is a detail of the implementation of something (whether that is your own code, or the core).  This will also mark that something as invisible for standard introspection:
````raku
class A {
    method foo() is implementation-detail { }
    method bar() { }
}
.name.say for A.^methods; # bar␤BUILDALL␤
````
Subroutines and methods in the core that are considered to be an implementation-detail, have been marked as such.  This should make it more clear which parts of the Rakudo implementation are game, and which parts are off-limits for developers (knowing that they can be changed without notice).  Yet another way to make sure that any Raku programs will continue to work with future versions of the Raku Programming Language.

## Invisible highlights


Ecosystem / Raku Land

CCR

IRC Logs


