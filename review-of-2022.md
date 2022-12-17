# Looking back

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
NYI "Frobnication";
```

### chown

The `chown` subroutine takes zero or more filenames, and changes the UID (with the `:uid` argument) and/or the GID (with the `:gid` argument) if possible.  Returns the filenames that were successfully changed.  There is also a `IO::Path.chown` method version.
``` 
my @files = ...;
say "Converted UID of {chown @files, :$uid} / @files.elems() files";
```

### head, skip, tail

The [.head](https://docs.raku.org/routine/head#(Any)_method_head), [.skip](https://docs.raku.org/routine/skip#(Any)_method_skip) and [.tail](https://docs.raku.org/routine/tail#(Any)_method_tail) methods go their subroutine counterparts._
```
say head 3, ^10;  # (0 1 2)
say skip 3, ^10;  # (3,4,5,6,7,8,9)
say tail 3, ^10;  # (7 8 9)
```

## New methods

### Any.are

The `.are` method returns the type object that all of the values of the invocant have in common.  This could be either a class or a role.
```
say (1,42e0,.137).are;         # (Real)
say (1,42e0,.137, "foo").are;  # (Cool)
say (42,DateTime.now).are;     # (Any)
```

### IO::Path.inode|dev|devtype|created|chown

The `IO::Path` class had 4 methods added:
- inode - the inode of the path (if available)
- dev - the device number of the filesystem (if available)
- devtype - the device identifier of the filesystem (if available)
- created - DateTime object
- chown - change uid and/or gid of path (if possible)


New:
roundrobin(..., :slip)
.chomp($needle)
Date(|Time).new(Y,M,\*)
Date(|Time).new(Y,M,\*-2)
Date(|Time).days-in-year
DateTime.posix(:real)
RAKUDO_MAX_THREADS
ThreadPoolScheduler.new(:max_threads(\*|Inf|unlimited))

Additions in 6.e:
.snip 
nano
.skip(produce, skip,...)
// as prefix
.snitch
.comb(rotor-args)

New experimental features:
will-complain
rakuast

Possibly breaking change:
Handling of Junctions in .classify / .categorize
$?COMPILATION-ID removed -> Compiler.id
.head|tail on native arrays return native arrays of same type rather than Seq
bare sort is a runtime error

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
