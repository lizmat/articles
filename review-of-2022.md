# 2022 Review

In a year as eventful as it was in the real world, it is a good idea to look back to see what one might have missed while life was messing with your Raku plans.  Unless otherwise noted, all of these changes are in language level 6.d, and available since a Rakudo compiler release in 2022.

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
my uint $foo = 42;
```

## New subroutines

### NYI

The `NYI` subroutine takes a string to indicate a feature not yet implemented, and turn that into a `Failure` with the `X::NYI` exception at its core.  You could consider this short for [`...`](https://docs.raku.org/routine/....html#(Operators)_listop_...) **with** feedback, rather than just the "Stub code executed".
```
say NYI "Frobnication";  # Frobnication not yet implemented. Sorry.
```

### chown

The `chown` subroutine takes zero or more filenames, and changes the UID (with the `:uid` argument) and/or the GID (with the `:gid` argument) if possible.  Returns the filenames that were successfully changed.  There is also a `IO::Path.chown` method version.
``` 
my @files = ...;
say "Converted UID of {chown @files, :$uid} / @files.elems() files";
```

### head, skip, tail

The [.head](https://docs.raku.org/routine/head#(Any)_method_head), [.skip](https://docs.raku.org/routine/skip#(Any)_method_skip) and [.tail](https://docs.raku.org/routine/tail#(Any)_method_tail)_ methods go their subroutine counterparts.
```
say head 3, ^10;  # (0 1 2)
say skip 3, ^10;  # (3,4,5,6,7,8,9)
say tail 3, ^10;  # (7 8 9)
```

## New methods

### Any.are

The `.are` method returns the type object that all of the values of the invocant have in common.  This could be either a class or a role.
```
say (1, 42e0, .137).are;         # (Real)
say (1, 42e0, .137, "foo").are;  # (Cool)
say (42, DateTime.now).are;      # (Any)
```
In some languages this appears to be called `infer`, but this name was deemed to be too ComputerSciency for Raku.

### IO::Path.inode|dev|devtype|created|chown

The `IO::Path` class had 4 methods added:
- inode - the inode of the path (if available)
- dev - the device number of the filesystem (if available)
- devtype - the device identifier of the filesystem (if available)
- created - DateTime object when path got created (if available)
- chown - change uid and/or gid of path (if possible)


### (Date|DateTime).days-in-year

A `.days-in-year` class method was added to the `Date` and `DateTime` classes.  It takes a year as positional argument:
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

### $*RAT-OVERFLOW

Determine the behaviour of rational numbers (aka [`Rat`s](https://docs.raku.org/type/Rat)) if they run out of precision.  More specifically when the denominator no longer fits in a native 64-bit integer.  By default, `Rat`s will be downgraded to floating point values (aka [`Num`s](https://docs.raku.org/type/Num)).  By setting the `$*RAT-OVERFLOW` dynamic variable, you can influence this behaviour.

The `$*RAT-OVERFLOW` is expected to contain an entity (usually a class) on which an `UPGRADE-RAT` method will be called.  This method is expected to take the numerator and denominator as positional arguments, and is expected to return whatever representation one wants for the given arguments.

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

### RAKUDO_MAX_THREADS

This environment variable can be set to indicate the **maximum** number of OS-threads that Rakudo may use for its thread pool.  The default is 64, or the number of CPU-cores times 8, whichever is larger.  Apart from a numerical value, you can also specify `"Inf`" or `"unlimited"` to indicate that Rakudo should use as many OS-threads as it can.

These same values can also be used in a call to `ThreadPoolScheduler.new` with the `:max_threads` named argument.

## New experimental features

### will complain

If you add a `use experimental :will-complain` to your code, you can customize typecheck errors by specifying a `will complain` trait.  The trait expects a `Callable` that will be given the offending value in question, and is expected to return a string to be added to the error message:
```
use experimental :will-complain;
my Int $a will complain { "You cannot use -$_-, dummy!" }
$a = "foo";
# Type check failed in assignment to $a; You cannot use -foo-, dummy!
```
The `will complain` trait can be used anywhere you can specify a type in Raku

### rakuast

If you add a `use experimental :rakuast` to your code, you will be able to use all of the `RakuAST` classes to build code programmatically without needing to create source code.  This is an entire new area of Raku development, which will be covered by many blog posts in the coming year.

For the **very** curious, you can check out a proof-of-concept in the new [`Formatter`](https://github.com/rakudo/rakudo/blob/main/src/core.e/Formatter.pm6) class, that builds executable code out of an [`sprintf` format](https://docs.raku.org/routine/sprintf#Directives).

## New arguments to existing functionality

### roundrobin(..., :slip)

The [`roundrobin`](https://docs.raku.org/routine/roundrobin) now also accepts a `:slip` named argument.  When specified, it will produce all values as a single, flattened list.
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



Date(|Time).new(Y,M,\*)
Date(|Time).new(Y,M,\*-2)

Additions in 6.e:
.snip 
nano
.skip(produce, skip,...)
// as prefix
.snitch
.comb(rotor-args)

Possibly breaking change:
Handling of Junctions in .classify / .categorize
$?COMPILATION-ID removed -> Compiler.id

Changed semantics in 6.e:
Int.roll
Int.pick

Removals:
RESTRICTED.setting

Unmentioned:
CompUnit::Repository::Staging.remove-artifacts
CompUnit::Repository::Staging.self-destruct
CompUnit::Repository::Staging.deploy
CompUnit::Repository::Installation.install(:!precompile)
Label.file|line
Cool|Exception.Failure coercer
Cool.Order coercer
Allow semicolon in my :($a,$b) = 42,666
Lock::Soft
INSIDE_EMACS environment variable
.hyper|batch Any is default
bare sort is a runtime error
.head|tail on native arrays return native arrays of same type rather than Seq
