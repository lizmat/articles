# Rakudo 2022 Review

In a year as eventful as it was in the real world, it is a good idea to look back to see what one might have missed while life was messing with your Raku plans.

Rakudo saw about 1500 commits this year, about the same as the year before that.  Many of these were bug fixes and performance improvements, which you would normally not notice.  But there were also commits that actually *added* features to the Raku Programming Language.  So it feels like a good idea to actually mention those more in depth.

So here goes!  Unless otherwise noted, all of these changes are in language level 6.d, and available since a Rakudo compiler release in 2022.

## New REPL functionality

It is now possible to refer to values that were produced earlier, using the `$*N` syntax, where `N` is a number greater than or equal to 0.
```
$ raku
To exit type 'exit' or '^D'
[0] > 42
42
[1] > 666
666
[2] > $*0 + $*1
708
```
Note that the number before the prompt indicates the index with which the value that is going to be produced, can be obtained.

## New MAIN options

You can now affect the interpretation of command line arguments to [`MAIN`](https://docs.raku.org/language/create-cli#index-entry-MAIN) by setting these options in the `%*SUB-MAIN-OPTS` hash:

### allow-no

Allow negation of a named argument to be specified as `--no-foo` instead of `--/foo`.

### numeric-suffix-as-value

Allow specification of a numeric value together with the name of a single letter named argument. So `-j2` being the equivalent of `--j=2`.

## New types

Native unsigned integers (both in scalar, as well as a (shaped) array) have become first class citizens.  This means that a native unsigned integer can now hold the value 18446744073709551615 as the largest positive value, from 9223372036854775807 before.  This also allowed for a number of internal optimisations as the check for negative values could be removed.  As simple as this sounds, this was quite an undertaking to get support for this on all VM backends.
```
my uint  $foo = 42;
my uint8 $bar = 255;
my  int8 $baz = 255;

say $foo;  # 42
say $bar;  # 255
say $baz;  # -1

say ++$foo;  # 43
say ++$bar;  # 0
say ++$baz;  # 0
```
And yes, all of the other explicitly sized types, such as `uint16`, `uint32` and `uint64`, are now also supported!

## New subroutines

A number of subroutines entered the global namespace this year.  Please note that they will not interfere with any subroutines in your code with the same name, as these always take precedence.

### NYI()

The `NYI` subroutine takes a string to indicate a feature not yet implemented, and turn that into a `Failure` with the `X::NYI` exception at its core.  You could consider this short for [`...`](https://docs.raku.org/routine/....html#(Operators)_listop_...) **with** feedback, rather than just the "Stub code executed".
```
say NYI "Frobnication";  # Frobnication not yet implemented. Sorry.
```

### chown()

The `chown` subroutine takes zero or more filenames, and changes the UID (with the `:uid` argument) and/or the GID (with the `:gid` argument) if possible.  Returns the filenames that were successfully changed.  There is also a `IO::Path.chown` method version.
``` 
my @files = ...;
say "Converted UID of {chown @files, :$uid} / @files.elems() files";
```

### head(), skip(), tail()

The [.head](https://docs.raku.org/routine/head#(Any)_method_head), [.skip](https://docs.raku.org/routine/skip#(Any)_method_skip) and [.tail](https://docs.raku.org/routine/tail#(Any)_method_tail)_ methods got their subroutine counterparts.
```
say head 3, ^10;  # (0 1 2)
say skip 3, ^10;  # (3,4,5,6,7,8,9)
say tail 3, ^10;  # (7 8 9)
```
Note that the number of elements is always the first positional argument.

## New methods

### Any.are

The `.are` method returns the type object that **all** of the values of the invocant have in common.  This could be either a class or a role.
```
say (1, 42e0, .137).are;         # (Real)
say (1, 42e0, .137, "foo").are;  # (Cool)
say (42, DateTime.now).are;      # (Any)
```
In some languages this appears to be called [`infer`](https://en.wikipedia.org/wiki/Inference), but this name was deemed to be too ComputerSciency for Raku.

### IO::Path.inode|dev|devtype|created|chown

Some low level IO features were added to the [`IO::Path`](https://docs.raku.org/type/IO::Path) class, in the form of 5 new methods.  Note that they may not actually work on your OS and/or filesystem (looking at you there, Windows :-)

- .inode - the inode of the path (if available)
- .dev - the device number of the filesystem (if available)
- .devtype - the device identifier of the filesystem (if available)
- .created - DateTime object when path got created (if available)
- .chown - change uid and/or gid of path (if possible, method version of `chown()`)

### (Date|DateTime).days-in-year

The [`Date`](https://docs.raku.org/type/Date) and [`DateTime`](https://docs.raku.org/type/DateTime) classes already provide many powerfule date and time manipulation features.  But a few features were considered missing this year, and so they were added.

A new `.days-in-year` class method was added to the `Date` and `DateTime` classes.  It takes a year as positional argument:
```
say Date.days-in-year(2023);  # 365
say Date.days-in-year(2024);  # 366
```
This behaviour was also expanded to the `.days-in-month` method, when called as a class method:
```
say Date.days-in-month(2023, 2);  # 28
say Date.days-in-month(2024, 2);  # 29
```
They can also be called as instance methods, in which case the parameters default to the associated values in the object:
```
given Date.today {
    .say;                # 2022-12-25
    say .days-in-year;   # 365
    say .days-in-month;  # 31
}
```

## New Dynamic Variables

[Dynamic variables](https://docs.raku.org/language/variables#index-entry-twigil_$*) provide a very powerful way to keep "global" variables.  A number of them are provided by the Raku Programming Language.  And now there is one more of them!

### $*RAT-OVERFLOW

Determine the behaviour of rational numbers (aka [`Rat`](https://docs.raku.org/type/Rat)s) if they run out of precision.  More specifically when the denominator no longer fits in a native 64-bit integer.  By default, `Rat`s will be downgraded to floating point values (aka [`Num`](https://docs.raku.org/type/Num)s).  By setting the `$*RAT-OVERFLOW` dynamic variable, you can influence this behaviour.

The `$*RAT-OVERFLOW` is expected to contain a class (or an object) on which an `UPGRADE-RAT` method will be called.  This method is expected to take the numerator and denominator as positional arguments, and is expected to return whatever representation one wants for the given arguments.

The following values can be specified using core features:

- Num

Default.  Silently convert to floating point.  Sacrifies precision for speed.

- CX::Warn

Downgrade to floating point, but issue a warning.  Sacrifies precision for speed.

- FatRat

Silently upgrade to [`FatRat`](https://docs.raku.org/type/FatRat), aka rational numbers with arbitrary precision.  Sacrifies speed by conserving precision.

- Failure

Return an appropriate `Failure` object, rather than doing a conversion.  This will most likely throw an exception unless specifically handled.

- Exception

Throw an appropriate exception.

Note that you can introduce any custom behaviour by creating a class with a `UPGRADE-RAT` method in it, and setting that class in the `$*RAT-OVERFLOW` dynamic variable.
```
class Meh {
    method UPGRADE-RAT($num,$denom) is hidden-from-backtrace {
        die "$num / $denom is meh"
    }
}
my $*RAT-OVERFLOW = Meh;
my $a = 1 / 0xffffffffffffffff;
say $a;      # 0.000000000000000000054
say $a / 2;  # 1 / 36893488147419103230 is meh
```

## New Environment Variables

Quite a few environment variables are already checked by Rakudo.  Two more were added in the past year:

### RAKUDO_MAX_THREADS

This environment variable can be set to indicate the **maximum** number of OS-threads that Rakudo may use for its thread pool.  The default is 64, or the number of CPU-cores times 8, whichever is larger.  Apart from a numerical value, you can also specify `"Inf`" or `"unlimited"` to indicate that Rakudo should use as many OS-threads as it can.

These same values can also be used in a call to `ThreadPoolScheduler.new` with the `:max_threads` named argument.
```
my $*SCHEDULER = ThreadPoolScheduler.new(:max_threads<unlimited>);
```

### INSIDE_EMACS

This environment variable can be set to a true value if you do **not** want the REPL to check for installed modules to handle editing of lines.  When set, it will fallback to the behaviour as if none of the supported line editing modules are installed.  This appears to be [handy for Emacs users](https://www.gnu.org/software/emacs/manual/html_node/emacs/Interactive-Shell.html), as the name implies :-)

## New experimental features

Some Raku features are not yet cast in stone yet, so there's no guarantee that any code written by using these experimental features, will continue to work in the future.  Two new experimental features have been added in the past year:

### :will-complain

If you add a `use experimental :will-complain` to your code, you can customize typecheck errors by specifying a `will complain` trait.  The trait expects a `Callable` that will be given the offending value in question, and is expected to return a string to be added to the error message.  For example:
```
use experimental :will-complain;
my Int $a will complain { "You cannot use -$_-, dummy!" }
$a = "foo";
# Type check failed in assignment to $a; You cannot use -foo-, dummy!
```
The `will complain` trait can be used anywhere you can specify a type constraint in Raku, so that includes parameters and attributes.

### :rakuast

The RakuAST classes allow you to dynamically build an AST ([Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) programmatically, and have that converted to executable code.  What was previously only possible by programmatically creating a piece of Raku code, and then calling [`EVAL`](https://docs.raku.org/routine/EVAL) on it.  But not only does it allow you to build code programmatically, it also allows you to introspect the AST, which opens up all sorts of syntax / lintifying possibilities.

There is an associated effort to compile the Raku core itself using a grammar that uses RakuAST to build executable code.  This effort is now capable of passing 585/1355 test-files in roast completely, and 83/131 of the Rakudo test-files completely.

So, if you add a `use experimental :rakuast` to your code, you will be able to use all of the `RakuAST` classes to build code programmatically.  This is an entire new area of Raku development, which will be covered by many blog posts in the coming year.  A small example, showing how to build the expression `"foo" ~ "bar"`:
```
use experimental :rakuast;

my $left  = RakuAST::StrLiteral.new("foo");
my $infix = RakuAST::Infix.new("~");
my $right = RakuAST::StrLiteral.new("bar");

my $ast = RakuAST::ApplyInfix.new(:$left, :$infix, :$right);
say $ast.DEPARSE;  # "foo" ~ "bar"
```
Note how each element of the expression can be created separately, and then combined together.  And that there is a `.DEPARSE` method to (re-)create the associated Raku source code.

For the **very** curious, you can check out a proof-of-concept of the use of `RakuAST` classes in the Rakudo core in the [`Formatter`](https://github.com/rakudo/rakudo/blob/main/src/core.e/Formatter.pm6) class, that builds executable code out of an [`sprintf` format](https://docs.raku.org/routine/sprintf#Directives).

## New arguments to existing functionality

### roundrobin(..., :slip)

The [`roundrobin`](https://docs.raku.org/routine/roundrobin) subroutine now also accepts a `:slip` named argument.  When specified, it will produce all values as a single, flattened list.
```
say roundrobin (1,2,3), <a b c>;         # ((1 a) (2 b) (3 c))
say roundrobin (1,2,3), <a b c>, :slip;  # (1 a 2 b 3 c)
```
This is functionally equivalent to:
```
say roundrobin((1,2,3), <a b c>).map: *.Slip;
```
but many times more efficient.

### Cool.chomp($needle)

The [`.chomp`](https://docs.raku.org/routine/chomp) method by default any logical newline from the end of a string.  It is now possible to specify a specific needle as a positional argument: only when that is equal to the end of the string, will it be removed.
```
say "foobar".chomp("foo");  # foobar
say "foobar".chomp("bar");  # foo
```
It actually works on all [`Cool`](https://docs.raku.org/type/Cool) values, but the return value will always be a string:
```
say 427.chomp(7);  # 42
```

### DateTime.posix

A [`DateTime`](https://docs.raku.org/type/DateTime) value has better than millisecond precision.  Yet, the [`.posix` method](https://docs.raku.org/type/DateTime#method_posix) always returned an integer value.  Now it can also return a `Num` with the fractional part of the second by specifying the `:real` named argument.
```
given DateTime.now {
    say .posix;         # 1671733988
    say .posix(:real);  # 1671733988.4723697
}
```

## Additional meaning to existing arguments

### Day from end of month

The `day` parameter to `Date.new` and `DateTime.new` (whether named or positional) can now be specified as either a `Whatever` to indicate the last day of the month, or as a `Callable` indicating number of days from the end of the month.
```
say Date.new(2022,12,*);    # 2022-12-31
say Date.new(2022,12,*-6);  # 2022-12-25
```

## Additions in v6.e.PREVIEW:

### term nano

A `nano` term is now available.  It returns the number of **nanoseconds** since midnight UTC since 1 January 1970.  It is similar to the [`time` term](https://docs.raku.org/routine/time) but one billion times more accurate.  It is intended for *very* accurate timekeeping / logging.
```
use v6.e.PREVIEW;
say time;  # 1671801948
say nano;  # 1671801948827918628
```
With current 64-bit native unsigned integer precision, this should roughly be enough for another 700 years :-)

### prefix //

You can now use `//` as a prefix as well as an infix.  It will return whatever the `.defined` method returns on the given argument).
```
use v6.e PREVIEW;
my $foo;
say //$foo;  # False
$foo = 42;
say //$foo;  # True
```
So you could consider `//$foo` to be syntactic sugar for `$foo.defined`.

### snip() and Any.snip

The new `snip` subroutine and method allows one to cut up a list into sublists according the given specification.  The specification consists of one or more smartmatch targets.  Each value of the list will be smartmatched with the given target: as soon as it returns `False`, will all the values before that be produced as a `List`.
```
use v6.e.PREVIEW;
say (2,5,13,9,6,20).snip(* < 10);
# ((2 5) (13 9 6 20))
```
Multiple targes can be specified.
```
say (2,5,13,9,6,20).snip(* < 10, * < 20);
# ((2 5) (13 9 6) (20))
```
The arguments can also be an `Iterable`.  To split a list consisting of integers and strings into sublists of just integers and just strings, you can do:
```
say (2,"a","b",5,8,"c").snip(|(Int,Str) xx *);
# ((2) (a b) (5 8) (c))
```
Inspired by Haskell's [`span`](https://hackage.haskell.org/package/base-4.16.1.0/docs/Prelude.html#v:span) function.

### Any.snitch

The new `.snitch` method is a debugging tool that will show its invocant with `note` by default, and return the invocant.  So you can insert a `.snitch` in a sequence of method calls and see what's happening "half-way" as it were.
```
$ raku -e 'use v6.e.PREVIEW; say (^10).snitch.map(*+1).snitch.map(* * 2)'
^10
(1 2 3 4 5 6 7 8 9 10)
(2 4 6 8 10 12 14 16 18 20)
```
You can also insert your own "reporter" in there: the `.snitch` method takes a `Callable`.  An easy example of this, is using `dd` for snitching:
```
$ raku -e 'use v6.e.PREVIEW; say (^10).snitch(&dd).map(*+1).snitch(&dd).map(* * 2)'
^10
(1, 2, 3, 4, 5, 6, 7, 8, 9, 10).Seq
(2 4 6 8 10 12 14 16 18 20)
```

### Any.skip(produce,skip,...)

You can now specify more than one argument to the `.skip` method.  Before, you could only specify a single (optional) argument.
```
my @a = <a b c d e f g h i j>;
say @a.skip;       # (b c d e f g h i j)
say @a.skip(3);    # (d e f g h i j)
say @a.skip(*-3);  # (h i j)
````
On v6.e.PREVIEW, you can now specify any number of arguments in the order: produce, skip, produce, etc.  Some examples:
```
use v6.e.PREVIEW;
my @a = <a b c d e f g h i j>;
# produce 2, skip 5, produce rest
say @a.skip(2, 5);        # (a b h i j)
# produce 0, skip 3, then produce 2, skip rest
say @a.skip(0, 3, 2);     # (d e)
# same, but be explicit about skipping rest
say @a.skip(0, 3, 2, *);  # (d e)
```
In fact, any `Iterable` can now be specified as the argument to `.skip`.
```
my @b = 3,5;
# produce 3, skip 5, then produce rest
say @a.skip(@b);           # (a b c i j)
# produce 1, then skip 2, repeatedly until the end
say @a.skip(|(1,2) xx *);  # (a d g j)
```

### Cool.comb(Pair)

On v6.e.PREVIEW, the [`.comb` method](https://docs.raku.org/routine/comb#(Str)_routine_comb) will also accept a `Pair` as an argument to give it [`.rotor`](https://docs.raku.org/routine/rotor#(Any)_method_rotor)_-like capabilities.  For instance, to produce [trigrams](https://en.wikipedia.org/wiki/Trigram) of a string, one can now do:
```
use v6.e.PREVIEW;
say "foobar".comb(3 => -2);  # (foo oob oba bar)
```
This is the functional equivalent of `"foobar".comb.rotor(3 => -2)>>.join`, but about 10x as fast.

### Changed semantics on Int.roll|pick

To pick a number from 0 till N-1, one no longer has to specify a range, but can just the integer value as the invocant:
```
use v6.e.PREVIEW;
say (^10).roll;     # 5
say 10.roll;        # 7
say (^10).pick(*);  # (2 0 6 9 4 1 5 7 8 3)
say 10.pick(*);     # (4 6 1 0 2 9 8 3 5 7)
```
Of course, all of these values are examples, as each run will, most likely, produce different results.

## The rest

There were some more new things and changes the past year.  I'll just mention them **very** succinctly here:

- New methods on `CompUnit::Repository::Staging`

`.deploy`, `.remove-artifacts`, and `.self-destruct`.

- `:!precompile` flag on `CompUnit::Repository::Installation.install`

Install module but precompile on first invocation rather than at installation.

- New methods on `Label`

`.file` and `.line` where the `Label` was created.

- .Failure coercer

Convert a `Cool` object or an `Exception` to a `Failure`.  Mainly intended to reduce binary size of hot paths that do some error checking.

- `Cool.Order` coercer

Coerce the given value to an `Int`, then convert to `Less` if less than 0, to `Same` if 0, and `More` if more than 0.

- Allow semi-colon

Now allow for the semi-colon in `my :($a,$b) = 42,666` because the left-hand side is really a `Signature` rather than a `List`.

## Summary

And about 1900 module releases, up from about 1650 in 2021.

I guess we've seen **one** big change in the past year, namely having experimental support for `RakuAST` become available.  And many smaller goodies and tweaks and features.  Now that `RakuAST` has become "mainstream" as it were, we can think of having certain optimizations.  Such as making `sprintf` with a fixed format string about 30x as fast!  Exciting times ahead!
