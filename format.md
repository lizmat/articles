# Moving printf formats forward

One of the advantages of [RakuAST](https://dev.to/lizmat/rakuast-for-early-adopters-576n) is that you can build executable code in an object oriented style.  One of the areas that has had the attention of yours truly in that respect, is handling of [`sprintf`](https://docs.raku.org/type/independent-routines#routine_sprintf) formats.

Historically the C programming language provided the first implementation of [`printf`](https://en.wikipedia.org/wiki/Printf) logic:

> The printf family of functions in the C programming language are a set of functions that take a format string as input among a variable sized list of other values and produce as output a string that corresponds to the format specifier and given input values.

## And in Raku?

Of course, the Raku Programming Language also has an implementation of `printf`, with a set of [supported options](https://docs.raku.org/type/independent-routines#Directives).  However the Rakudo implementation of the Raku Programming Language does not have an implementation of `printf` formats of its own, but borrows it from [`NQP`](https://github.com/raku/nqp#readme).  Which has its pros and its cons.

The pros are of course being that NQP in general is much faster than Raku (as NQP is sort of the ["assembly language"](https://en.wikipedia.org/wiki/Assembly_language) of Rakudo).  However, the NQP implementation of `sprintf` is written using a grammar (written in NQP).  And **each** time a call is made to this `sprintf` functionality (e.g, when printing a line in a report using `sprintf`), the whole grammar is run again, with the associated actions building the string to be returned.  And running a grammar is costly in terms of memory *and* CPU.

This makes Rakudo's `sprintf` and associated functions (such as [`.fmt`](https://docs.raku.org/routine/fmt)) one of the slowest features of Raku in Rakudo.  Surely with RakuAST there must be a better way to do this?

## Enter Formatter

Almost two years ago, yours truly started on the RakuAST journey.  First by writing tests, later trying out RakuAST features.  And one of the first things attempted, was to port the original NQP grammar for `sprintf` to Raku.  And create a separate set of actions, that would **not** produce a string, but instead would produce an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree).

Such an AST can then be converted to executable code, and then run whenever a set of arguments would need to be converted to a string.  So instead of needing to run the same grammar over and over again, it would be possible to just call some (static) code, which then, like any other piece of code in Raku, would get run-time optimized for better performance.

This then became the [`Formatter`](https://github.com/rakudo/rakudo/blob/main/src/core.e/Formatter.pm6) class.  And since this was a completely new feature, it only became available for use by opting into the `6.e.PREVIEW` language version.  And then it went largely unnoticed and uncared for the next 1.5 year.  As clearly the time wasn't right for it yet.

## Obstacles

One of the obstacles that was encountered while working on `Formatter`, was that there were *no* comprehensive tests for `sprintf` functionality for Raku.  Yes, there as *a* test-file, that tested some features, but no comprehensive tests for all possible combinations of format features, combined with possible values that they should operate upon.

This became one of the first things that needed to be done, to be able to say that new implementation would be matching the old.  During the development of [these tests](https://github.com/Raku/roast/tree/master/6.d/S32-str), it became clear there were some inconsistencies in the existing implementation, and worse: [outright bugs](https://github.com/Raku/roast/blob/master/6.d/S32-str/sprintf-f.t#L101).

So the question became: should the new implementation follow the behaviour of the old implementation to the letter, or not?

## Raku Core Summit

This then became part of discussions at the first Raku Core Summit.  There it was decided that the new implementation should not adhere to the old one, because the new `sprintf` functionality would require a language version bump anyway.  And that the current `sprintf` tests should be frozen for language level `6.d`.  And so it was [done](https://github.com/Raku/roast/commit/a297e8d4e2510e0fbef2cbd4c766d5e4927f029f).

Another decision that was made, was that the functionality offered by the `Formatter` class (which converts a format string to a [`Callable`](https://docs.raku.org/type/Callable), should be embedded into a [`Format`] class.  Which should act as a string, but when used as a `Callable`, should perform the processing according to the format given.  This has now also been [implemented](https://github.com/rakudo/rakudo/commit/ebe0e0b2c7290dd27729a71da65e55f3f3a72558).  Which means you can now do:
```
use v6.e.PREVIEW;
my $f = Format.new("%5s");
dd $f;              # Format.new("%5s")
say $f;             # %5s
dd $f("foo");       # "  foo"
say "'$f'";         # '%5s'
say "'$f("foo")'";  # '  foo'
```
As you can see, the `Format` object acts very much so as a string.  Only when used as a `Callable` (by putting `()` after it, with arguments, will it show its special functionality.  Which isn't so strange when you realize the method resolution order of `Format`:
```
use v6.e.PREVIEW;
say Format.^mro;  # ((Format) (Str) (Cool) (Any) (Mu))
```

## A new quote adverb

At the RCS it was suggested that the `Format.new("%5s") would be too clunky.  But more seriously, would prevent compile time optimization of the `Format` object, because the lookup of methods are a runtime action in Raku (generally).  A way to get around that, would be to introduce a new string quoting construct / adverb / processor.  This [became](https://github.com/rakudo/rakudo/commit/e95c45a5ec641c301f0eec9c371f36683a0496fd) the `"format"` adverb (or `"o"` for the short version):
```
use v6.e.PREVIEW;
my $format = q:format/%5s/;
say "'$format("foo")'";   # '  foo'
```
or more directly:
```
use v6.e.PREVIEW;
dd q:format/%5s/("foo");  # "  foo"
```
Why `"o"` for the short version?  Because `"f"` was already taken to enable/disable *f*unction interpolation.  And `q:o` felt visually closer to `fo` than anything else.

## Performance
So how does this new way of formatting values into a string perform?  Without any optimization of the RakuAST version yet, upto a 30x speed increase has been reported.  So that's quite an improvement.  And potentially better than many other programming languages.

## Conclusion
A new way of creating strings from a given set of values and format string (in other words `printf` functionality) has been implemented in the Raku Programming Language, making it up to 30x as fast.

This functionality will be available from release 2023.06 of Rakudo.  The new quote adverb will for now only be available when compiled with the new RakuAST grammar, which can be activated by specifying the `RAKUDO_RAKUAST` environment variable.
