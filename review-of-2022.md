

New:
Cool.Order coercer
Cool|Exception.Failure coercer
roundrobin(..., :slip)
List.are
native unsigned integers
NYI
.chomp($needle)
Label.file|line
Date(|Time).new(Y,M,\*)
Date(|Time).new(Y,M,\*-2)
Date(|Time).days-in-year
CompUnit::Repository::Staging.remove-artifacts
CompUnit::Repository::Staging.self-destruct
CompUnit::Repository::Staging.deploy
CompUnit::Repository::Installation.install(:!precompile)
INSIDE_EMACS environment variable
Accessing previous values in REPL with `$*N`
Allow semicolon in my :($a,$b) = 42,666
DateTime.posix(:real)
Lock::Soft
RAKUDO_MAX_THREADS
ThreadPoolScheduler.new(:max_threads(\*|Inf|unlimited))
IO::Path.inode|dev|devtype|chown|created
chown()
Sub version of head | skip | tail
.hyper|batch Any is default
%\*SUB-MAIN-OPTS = :allow-no
%\*SUB-MAIN-OPTS = :numeric-suffix-as-value

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
