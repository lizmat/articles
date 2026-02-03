# Catch Control

> This is part six in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the phasers that catch exceptions of various kinds.

## CATCH

Many programming languages have a `try` / `catch` mechanism.  Although it is true that the [Raku Programming Language](https://raku.org) does have a [`try` statement prefix](https://docs.raku.org/syntax/try%20%28statement%20prefix%29), and it does have a [`CATCH`](https://docs.raku.org/syntax/CATCH#Resuming_of_exceptions) you should generally **not** use both.

In Raku **any** scope can have a **single** `CATCH` block.  The code within it will be executed as soon as any runtime exception occurs in that scope, with the exception that was thrown topicalized in [`$_`](https://docs.raku.org/syntax/%24_).

> This is the reason you cannot have a `CATCH` thunk: it needs to have a scope to be able to set `$_` in there, without affecting anything outside of that scope.

It doesn't matter where in a scope you put the `CATCH` block.  But it is recommended for clarity's sake to put a `CATCH` block as early in the scope as possible (rather than "hiding" it somewhere near the end of a scope).

### Handling exceptions

Let's go again for a contrived example:
```raku
{ 
    CATCH {
        when X::AdHoc {
            say "naughty: $_.message()";
        }
    }
    die "urgh";
    say "after";
}
say "alive still";
```
Running the above code will show:
```
naughty: urgh
alive still
```
Note that having the exception in `$_` smart-match with [`when`](https://docs.raku.org/syntax/when) effectively disables the exception so it won't be re-thrown on scope exit.  Because of that, the `say "alive still"` will be executed.  Any other type of error would **not** be disabled, although you could if you wanted that with a [`default`](https://docs.raku.org/syntax/default%20when) block.

The careful reader will have noticed that the `say "after"` was **not** executed.  That's because even though the exception was disabled by the `when`, there was still an exception thrown.  And thatmeant that the current scope would be left pretty much as if a `return Nil` where executed.

If you feel like the exception in question is benign, you can execute the [`.resume`](https://docs.raku.org/type/Exception#method_resume) method on the exception object.  So let's assume all errors we get in that block are benign, and we want to just continue after each exception:
```raku
{ 
    CATCH {
        default {
            .resume
        }
    }
    die "urgh";
    say "after";
}
say "alive still";
```
would show:
```
after
alive still
```
However, not all exceptions are resumable!  So your program may stop nonetheless if the exception can not be actually resumed.

### Trying code

The `try` statements prefix (which either takes a thunk or a block) is actually a simplified case of a `CATCH` handler that disarms all errors, sets [`$!`](https://docs.raku.org/syntax/%24%21) and returns [`Nil`](https://docs.raku.org/type/Nil).  The code:
```raku
say try die "urgh";  # Nil
dd $!;               # $! = X::AdHoc.new(payload => "urgh")
```
is actually pretty much short for:
```raku
say {
    CATCH {
        CALLERS::<$!> = $_;  # set $! in right scope
        default { }          # disarm exception
    }
    die "urgh";
}();    # Nil
dd $!;  # $! = X::AdHoc.new(payload => "urgh")
```
Note that even though `die "urgh"` is a thunk, the compiler turned this into its own scope internally to be able to handle the return from the thunk.

## CONTROL

Apart from runtime exceptions, many other types of exceptions are used in Raku.  They all consume the [`X::Control` role](https://docs.raku.org/type/X/Control) and are therefore referred to as ["control exceptions"](https://docs.raku.org/language/exceptions#Control_exceptions).  They are used for the following Raku features (in alphabetical order):
- [`done`](https://docs.raku.org/routine/done) - call "done" callback on all taps
- [`emit`](https://docs.raku.org/routine/emit) - send item to all taps of supply
- [`last`](https://docs.raku.org/syntax/last) - exit the loop structure
- [`next`](https://docs.raku.org/syntax/next) - start next iteration in loop structure
- [`proceed`](https://docs.raku.org/language/control#proceed_and_succeed) - resume after `given` block
- [`redo`](https://docs.raku.org/syntax/redo) - restart iteration in loop structure
- [`return`](https://docs.raku.org/routine/return) - return from sub / method
- [`succeed`](https://docs.raku.org/syntax/succeed) - exit `given` block
- [`take`](https://docs.raku.org/routine/take) - pass item to `gather`
- [`warn`](https://docs.raku.org/routine/warn) - warn with given message

Just as runtime exceptions they all perform the work they are expected to do without needing any specific action from a user.  However, you can **not** catch these exceptions with a `CATCH` block, you need a [`CONTROL` block](https://docs.raku.org/syntax/CONTROL) for that.

### Handling control exceptions yourself

Suppose you have a pesky warning that you want to get rid of:
```raku
say $_ ~ "foo";
```
which would show:
```
Use of uninitialized value element of type Any in string context.
Methods .^name, .raku, .gist, or .say can be used to stringify it to something meaningful.
  in block foo at bar line 42
foo
```
You could do that using a `CONTROL` block like this:
```raku
CONTROL {
    when CX::Warn { .resume }
}
say $_ ~ "foo";
```
which would just show:
```
foo
```
Pretty simple, eh?   Well, by popular demand there is actually a shortcut for this case, called the [`quietly` statement prefix](https://docs.raku.org/syntax/quietly%20%28statement%20prefix%29):
```raku
quietly say $_ ~ "foo";
```
The standard way of handling warnings only produces the exact call-site where the warning occurred.  Which sometimes is just not enough to find the reason for the warning, because you would need a full stack trace for that.  Well, you can do that as well with a `CONTROL` block:
```raku
CONTROL {
    when CX::Warn {
        note .message;
        note .backtrace.join;
        .resume;
    }
}
say $_ ~ "foo"
```
would show:
```
Use of uninitialized value element of type Any in string context.
Methods .^name, .raku, .gist, or .say can be used to stringify it to something meaningful.
  in hsub warn at SETTING::src/core.c/control.rakumod line 267
  in method Str at SETTING::src/core.c/Mu.rakumod line 817
  in method join at SETTING::src/core.c/List.rakumod line 1200
  inw sub infix:<~> at SETTING::src/core.c/Str.rakumod line 3995
  in block foo at bar line 42
```
Or if you would like to turn any warning into a runtime exception:
```raku
CONTROL {
    when CX::Warn { .throw }
}
say $_ ~ "foo";
```
would show:
```
Use of uninitialized value $_ of type Any in string context.
Methods .^name, .raku, .gist, or .say can be used to stringify it to something meaningful.
  in block foo at bar line 42
```
and **not** show the "foo" because it would never get to that.

### Building your own control exceptions

Can you build you own control exceptions?  Yes, you can.  Are they useful?  Probably not.  But just in case someone finds a way to make this useful, here's how you do it.  You start with a class that consumes the `X::Control` role:
```raku
class Frobnicate does X::Control {
    has $.message;
}
```
Then you create a nice subroutine to make it easier to create an instance of that class, pass it any arguments and throw the instantiated control exception object:
```raku
sub frobnicate($message) {
    Frobnicate.new(:$message).throw
}
```
Then in your code, make sure there is a `CONTROL` phaser in the dynamic scope that handles the control exception:
```raku
CONTROL {
    when Frobnicate {
        say "Caught a frobnication: $_.message()";
        .resume;
    }
}
```
And then execute the nice subroutine at will:
```raku
say "before";
frobnicate "This";
say "after";
```
which would show:
```
before
Caught a frobnication: This
after
```

## Final notes

Even though the `CATCH` and `CONTROL` phasers are scope based, you can **not** introspect a given `Block` to see whether it has `CATCH` or `CONTROL` phasers.  This is because you can only have one of them in a scope, and handling exceptions in these phasers actually needs to be built into the bytecode generated for a block.  So from a runtime point of view, there was no need to put them as attributes into the `Block` object.

Exception handling in Raku is built on {delimited continuations](https://en.wikipedia.org/wiki/Delimited_continuation).  If you really want to get into the nitty gritty of this feature, you might be interested in reading ["Continuations in NQP"](https://github.com/Raku/nqp/blob/main/docs/continuations.pod#continuations-in-nqp).

## Conclusion

The `CATCH` catches exceptions that are supposed to be fatal.  The `CONTROL` phasers caches exceptions for all sorts of "normal" functionality, such as `next`, `last`, `warn`, etc.

You can build your own control exceptions, but the utility of these remains unclear as of yet.

This concludes the sixth episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
