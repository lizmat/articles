Perl 6 Phasers 101
==================

This is the sixth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the
[special blocks in Perl 5](https://perldoc.pl/perlmod#BEGIN,-UNITCHECK,-CHECK,-INIT-and-END)
such as `BEGIN` and `END`, and the possible subtle change in semantics with
so-called [phasers](https://docs.perl6.org/language/phasers) in Perl 6.

An overview
===========
Let's start with an overview again of Perl 5 special blocks and their Perl 6
counterparts, in the order they get executed in Perl 5:

| Perl 5    | Perl 6 | Notes      |
|:---------:|:-----------:|:----------:|
| BEGIN     | BEGIN  | not run when loading pre-compiled code |
| UNITCHECK | CHECK  | |
| CHECK     |        | no equivalent in Perl 6 |
| INIT      | INIT   | |
| END       | END    | |

These phasers in Perl 6 are usually referred to as
[Program Execution Phasers](https://docs.perl6.org/language/phasers#Program_execution_phasers).  This is because they are related to the execution of a
complete program, regardless of where they are located in a program.

BEGIN
=====
The semantics of the `BEGIN` special block in Perl 5 are exactly the same as
the [`BEGIN`](https://docs.perl6.org/language/phasers#BEGIN) phaser in Perl 6.
It specifies a piece of code to be executed **immediately** as soon as it has
been parsed (so *before* the program, aka the compilation unit as a whole has
been parsed).  There is however a caveat with the use of `BEGIN` in Perl 6!

Caveat when using BEGIN in modules in Perl 6
--------------------------------------------
Modules in Perl 6 are pre-compiled by default.  This means that the `BEGIN`
block is only executed whenever the (pre)compilation occurs, **not** every
time the module is loaded.  This is different from Perl 5, where modules
ordinarily only exist as source code that is compiled whenever a module is
loaded (even though that module can load already compiled native library
components).

This may cause some unpleasant surprises when porting code from Perl 5 to
Perl 6, because pre-compilation may have happened a long time ago, or even
on a different machine altogether (in case of having installed from an
OS distributed package).

An easy work-around would be to inhibit pre-compilation of a module in Perl 6:

    # Perl 6
    no precompilation;  # this code should not be pre-compiled

However, pre-compilation has a **very** big advantage: it can load modules
**much** faster because it doesn't need to parse any source code.  The prime
example of this is the core setting of Perl 6: the part of Perl 6 that is
actually written in Perl 6.  This currently exists of a 64KLOC / 2MB+ source
file (generated from many separate source files for maintainability).  It
takes about 1 minute to compile this source file during installation of
Perl 6.  It takes about 125 msecs to load this pre-compiled code at Perl 6
startup.  Which is almost a **500x** speedup!

Of course, you can also use this as a feature:

    # Perl 6
    say "This module was compiled at { BEGIN DateTime.now }";
    # This module was compiled at 2018-10-04T22:18:39.598087+02:00

Some other features of Perl 5 **and** Perl 6 that implicitly use `BEGIN`
functionality, suffer from the same caveat.  Take this example where we want
to have a constant `foo` to either have the value of the environment variable
`FOO`, or if that is not available, the value `42`:

    # Perl 5
    use constant foo => $ENV{FOO} // 42;

    # Perl 6
    my constant foo = %*ENV<FOO> // 42; 

The best equivalent in Perl 6 is probably using an `INIT` phaser:

    # Perl 6
    INIT my \foo = %*ENV<FOO> // 42;  # sigilless variable bound to value

But more about this syntax later.

UNITCHECK
=========
The functionality of the `UNITCHECK` special block in Perl 5 is performed by
the [`CHECK`](https://docs.perl6.org/language/phasers#CHECK) phaser in Perl 6.
Otherwise, it is exactly the same: it specifies a piece of code to be executed
when *compilation* of the current compilation unit is done.

CHECK
=====
There is **no** equivalent in Perl 6 of the `CHECK` special block of Perl 5.
The main reason for this is that you probably shouldn't be using the `CHECK`
special block in Perl 5 anymore, but use `UNITCHECK` instead as the `UNITCHECK`
semantics are much more sane (available since
[version 5.10](https://metacpan.org/pod/perl5100delta#UNITCHECK-blocks)).
Therefore this special block will not be further discussed in this article.

INIT
====
The functionality of the [`INIT`](https://docs.perl6.org/language/phasers#INIT)
phaser in Perl 6 is exactly the same as the `INIT` special block in Perl 5.
It specifies a piece of code to be executed **just before** the code in the
compilation unit will be executed.

In pre-compiled modules in Perl 6, the `INIT` phaser can serve as an
alternative to the `BEGIN` phaser.

END
===
The functionality of the [`END`](https://docs.perl6.org/language/phasers#END)
phaser in Perl 6 is exactly the same as the `END` special block in Perl 5.
It specifies a piece of code to be executed **after** all the code in the
compilation unit has been executed, or the code decides to exit (either
intended, or unintended because of an exception having been thrown).

Example
=======
Example using all four Program Execution Phasers and their Perl 5 special
block counterparts

    # Perl 5
    say "running in Perl 5";
    END       { say "END"   }
    INIT      { say "INIT"  }
    UNITCHECK { say "CHECK" }
    BEGIN     { say "BEGIN" }
    # BEGIN
    # CHECK
    # INIT
    # running in Perl 5
    # END

    # Perl 6
    say "running in Perl 6";
    END   { say "END"   }
    INIT  { say "INIT"  }
    CHECK { say "CHECK" }
    BEGIN { say "BEGIN" }
    # BEGIN
    # CHECK
    # INIT
    # running in Perl 6
    # END

More than special blocks
========================
Phasers in Perl 6 actually have some additional features that make them so
much more than just special blocks.

No need for a block
-------------------
Most phasers in Perl 6 do not have to be a
[Block](https://docs.perl6.org/type/Block) (aka, followed by code between
curly braces).  They can also consist of a single statement *without* any
curly braces.  This means that if you've written in Perl 5:

    # Perl 5
    # need to define lexical outside of BEGIN scope
    my $foo;
    # otherwise it won't be known in the rest of the code
    BEGIN { $foo = %*ENV<FOO> // 42 };

you can write this in Perl 6 as:

    # Perl 6
    # share scope with surrounding code
    BEGIN my $foo = %*ENV<FOO> // 42;

May return a value
------------------
All program execution phasers actually *return* the last value of their
code.  So you can actually use them in an expression.  The above example
using `BEGIN` can also be written as:

    # Perl 6
    my $foo = BEGIN %*ENV<FOO> // 42;

When used like that with a `BEGIN` phaser, you are in fact creating a
nameless constant and assigning that at runtime.

Because of pre-compilation of modules, if you would like this type of
initialization in a module, you would probably be better of using the
`INIT` phaser:

    # Perl 6
    my $foo = INIT %*ENV<FOO> // 42;

This will ensure that the value will be determined at the moment the module
is **loaded**, rather then whenever it was pre-compiled (which typically
only happens once during installation of the module).

Additional phasers in Perl 6
============================
Perl 6 has generalized some other features of Perl 5 as phasers, that weren't
covered by special blocks in Perl 5.  And it has added some more phasers that
are not covered by any (standard) functionality of Perl 5 at all.

> One important feature that one should remember about phasers is that a
> developer of a program does **not** determine **when** a phaser is run.
> It is the runtime executor that decides when a phaser is being run.
> Therefore all phasers in Perl 6 are spelled in uppercase characters,
> to make them stand out better as they are *not* part of the standard
> flow of execution.

Exception handling phasers
--------------------------
In Perl 5 you can use [`eval`](https://perldoc.perl.org/functions/eval.html)
to catch exceptions in a piece of code.  In Perl 6 this functionality is
covered by
[try](https://docs.perl6.org/language/exceptions#index-entry-try_blocks):

    # Perl 5
    eval {
        die "Goodbye cruel world";
    };
    say $@;           # Goodbye cruel world at ...
    say "Alive again!"

    # Perl 6
    try {
        die "Goodbye cruel world";
    }
    say $!;           # Goodbye cruel world␤  in block ...
    say "Alive again!"

In Perl 5 you can also use the return value of `eval` in an expression:

    # Perl 5
    my $foo = eval { ... };  # undef if exception was thrown

That works the same way in Perl 6 for `try`:

    # Perl 6
    my $foo = try { ... };   # Nil if exception was thrown

If you however need finer control over what to do when an exception occurs,
you can use special [signal handlers](https://perldoc.pl/variables/%25SIG)
`$SIG{__DIE__}` and `$SIG{__WARN__}` in Perl 5.

In Perl 6, these are replaced by two exception handling phasers, which due
to their scoping behaviour, *must* always be specified using curly braces.
They're only applicable to the surrounding block.  And you can only have
one of each type in a block.

| Name | Description |
|:----------|:-----------|
| CATCH   | run when an exception is thrown |
| CONTROL | run for any (other) control exception |

CATCH
-----
The use of `$SIG{__DIE__}` in Perl 5 is not recommended anymore.  Several
competing CPAN modules provide some kind of `try / catch` mechanism (such
as: [Try::Tiny](https://metacpan.org/pod/Try::Tiny) and
[Syntax::Keyword::Try](https://metacpan.org/pod/Syntax::Keyword::Try)).
However, there isn't really a single Perl 5 target to compare Perl 6 features
against.  So in this case only Perl 6 code will be shown.

The code inside a [`CATCH`](https://docs.perl6.org/language/phasers#CATCH)
phaser will be called whenever an exception is thrown in the immediately
surrounding lexical scope.

    # Perl 6
    {
        CATCH {
            say "aw, died";
            .resume;
        }
        die "goodbye cruel world";
        say "alive again";
    }
    # aw, died
    # alive again

Please note that you do **not** need a `try` statement to be able to catch
exceptions.  A `try` block is just a convenient way to disregard any
exceptions without having to worry about anything.

Also note that `$_` will be set to the `Exception` object inside the `CATCH`
block.  In this example, the `Exception` is simply resumed by calling the
`resume` method on the `Exception` object.  Execution will then continue
with the statement after the statement that caused the exception.  If the
exception is not resumed, it will be thrown again, to be possibly caught
by an outer `CATCH` block (if any).

Checking for specific exception is made easy with the
[`when`](https://docs.perl6.org/language/control#default_and_when) statement:

    # Perl 6
    {
        CATCH {
            when X::NYI {       # Not Yet Implemented exception thrown
                say "aw, too early in history";
                .resume;
            }
            default {
                say "WAT?";
                .rethrow;       # throw the exception again
            }
        }
        X::NYI.new(feature => "Frobnicator").throw;  # caught, resumed
        now / 0;                                     # caught, rethrown
        say "back to the future";
    }
    # aw, too early in history
    # WAT?
    # Attempt to divide 1234.5678 by zero using /

In this example only `X::NYI` exceptions will be resumed, all other ones will
be thrown to any outer `CATCH` block and as such will probably result in
program termination.

Catching warnings
-----------------
If you do not want any warnings to emanate from the execution of a piece of
code, you can use the `no warnings` pragma in Perl 5:

    # Perl 5
    {
        no warnings;
        my $foo;
        say $foo;     # no visible warning
    }
    my $bar;
    print $bar;
    # Use of uninitialized value $bar in print...

In Perl 6 you can use a `quietly` block:

    # Perl 6
    quietly {
        my $foo;
        say $foo;     # no visible warning
    }
    my $bar;
    print $bar;
    # Use of uninitialized value of type Any in string context...

The `quietly` block will catch **any** warnings that emanate from that block
and just disregards them.

If you want finer control on which warnings you want to be seen, you can
select which
[warning categories](https://perldoc.pl/warnings#Category-Hierarchy) you
want enabled / disabled with `use warnings` / `no warnings` in Perl 5.
In Perl 6 however, you would need a `CONTROL` phaser:

CONTROL
-------
The `CONTROL` phaser is very much like the `CATCH` phaser, but it handles
a special type of exception, the so-called "control exception".  Whenever
a warning is generated in Perl 6, a control exception is thrown.  Which
you can catch with the `CONTROL` phaser.  Here's an example that will not
show 

    # Perl 6
    CONTROL {
        when CX::Warn {    # control exception type associated with warnings
            note .message
              unless .message.starts-with('Use of uninitialized value');
        }
    }

Sadly, there are currently no warning categories defined in Perl 6.  So you
will have to check for the actual `message` of the control exception of
type `CX::Warn`.

The controle exception mechanism is also used for quite a lot of other
functionality apart from warnings.  The following statements also create
control exceptions (in alphabetical order): 
[emit](https://docs.perl6.org/language/control#supply/emit),
[fail](https://docs.perl6.org/language/control#fail),
[last](https://docs.perl6.org/language/control#last),
[next](https://docs.perl6.org/language/control#next),
[proceed](https://docs.perl6.org/language/control#proceed),
[redo](https://docs.perl6.org/language/control#redo),
[return](https://docs.perl6.org/language/control#return),
[return-rw](https://docs.perl6.org/language/control#return-rw),
[succeed](https://docs.perl6.org/language/control#succeed),
[take](https://docs.perl6.org/language/control#gather/take).  So control
exceptions generated by these statements will also show up in any `CONTROL`
phaser.  Luckily, if you do not do anything with the given control exception,
it will be rethrown when the `CONTROL` phaser is done.  And that will make
sure its intended action will be performed.

Block and Loop phasers
----------------------
Block and Loop phasers are always associated with the surrounding block,
regardless of where they are located in the block, just the way that the
`CATCH` and `CONTROL` phasers are.  Except you are not limited to only
having one of each: although you could argue that having more than one
doesn't improve maintainability.

| Name | Description |
|:----------|:-----------|
| ENTER | run everytime when entering a Block |
| LEAVE | run everytime when leaving a Block |
| PRE   | check condition *before* running a Block |
| POST  | check return value *after* having run a Block |
| KEEP  | run everytime a Block is left successfully |
| UNDO  | run everytime a Block is left **un**successfully |

ENTER & LEAVE
-------------
The [`ENTER`](https://docs.perl6.org/language/phasers#ENTER) and
[`LEAVE`](https://docs.perl6.org/language/phasers#LEAVE) phasers are pretty
self-explanatory: the `ENTER` phaser is called whenever a block is entered.
The `LEAVE` phaser is called whenever a block is left (either gracefully or
through an exception).  A simple example:

    # Perl 6
    say "outside";
    {
        LEAVE say "left";
        ENTER say "entered";
        say "inside";
    }
    say "outside again";
    # outside
    # entered
    # inside
    # left
    # outside again

The last value of an `ENTER` phaser is actually returned, so it can be
used in an expression.  A bit of a contrived example;

    # Perl 6
    {
        LEAVE say "stayed " ~ (now - ENTER now) ~ " seconds";
        sleep 2;
    }
    # stayed 2.001867 seconds

KEEP & UNDO
-----------
The [`KEEP`](https://docs.perl6.org/language/phasers#KEEP) and
[`UNDO`](https://docs.perl6.org/language/phasers#UNDO) phasers are special
cases of the `LEAVE` phaser.  They are called depending on the return value
of the surrounding block.  If the result of calling the
[`defined`](https://docs.perl6.org/type/Mu#index-entry-method_defined) method
on the return value is `True`, then any `KEEP` phasers will be called.  If
the result of calling `defined` is not `True`, then any `UNDO` phaser will be
called.  The actual value of the block will be available in the topic
(aka `$_`):

A contrived example may clarify:

    # Perl 6
    for 42, Nil {
        KEEP { say "Keeping because of $_" }
        UNDO { say "Undoing because of $_.perl()" }
        $_;
    }
    # Keeping because of 42
    # Undoing because of Nil

A bit of a real-life example:

    # Perl 6
    {
        KEEP $dbh.commit;
        UNDO $dbh.rollback;
        ...    # set up a big transaction in a database
        True;  # indicate success
    }

So if anything should go wrong with setting up the big transaction in the
database, the `UNDO` phaser will make sure that the transaction is rolled
back.  Conversely, if the block is successfully left, then the transaction
will be automatically committed by the `KEEP` phaser

The `KEEP` and `UNDO` phasers give you the building blocks for a poor man's
[Software Transactional Memory](https://en.wikipedia.org/wiki/Software_transactional_memory).

PRE & POST
----------
The [`PRE`](https://docs.perl6.org/language/phasers#PRE) phaser is a special
version of the `ENTER` phaser.  The
[`POST`](https://docs.perl6.org/language/phasers#POST) phaser is a special
case of the `LEAVE` phaser.

The `PRE` phaser is expected to return a true value if it is ok to enter
the block.  If it does not, then an exception will be thrown.  The `POST`
phaser receives the return value of the block and is expected to return
a true value if it is ok to leave the block without throwing an exception.

Some examples:

    # Perl 6
    {
        PRE { say "called PRE"; False }    # throws exception
        ...
    }
    say "we made it!";                     # never makes it here
    # called PRE
    # Precondition '{ say "called PRE"; False }' failed

    # Perl 6
    {
        PRE  { say "called PRE"; True   }  # does NOT throw exception
        POST { say "called POST"; False }  # throws exception
        say "inside the block";            # also returns True
    }
    say "we made it!";                     # never makes it here
    # called PRE
    # inside the block
    # called POST
    # Postcondition '{ say "called POST"; False }' failed

Loop phasers
------------
Loop phasers are a special type of Block phaser that are specific to loop
constructs: one that is
run before the first iteration
([FIRST](https://docs.perl6.org/language/phasers#FIRST)), one that is run
after each iteration
([NEXT](https://docs.perl6.org/language/phasers#NEXT)), and one that is run
after the last iteration ([LAST](https://docs.perl6.org/language/phasers#LAST)):

| Name | Description |
|:----------|:-----------|
| FIRST | run only the first time through a loop |
| NEXT  | run after each completed iteration, or with next |
| LAST  | run after the last iteration, or with last |

The names really speak for themselves.  A bit of a contrived example:

    # Perl 6
    my $total = 0;
    for 1..5 {
        $total += $_;
        LAST  say "------ +\n$total.fmt('%6d')";
        FIRST say "values\n======";
        NEXT  say .fmt('%6d');
    }
    # values
    # ======
    #      1
    #      2
    #      3
    #      4
    #      5
    # ------ +
    #     15

Loop constructs include [`loop`](https://docs.perl6.org/language/control#loop),
[`while`, `until`](https://docs.perl6.org/language/control#while,_until),
[`repeat/while`, `repeat/until](https://docs.perl6.org/language/control#repeat/while,_repeat/until),
[`for`](https://docs.perl6.org/language/control#for) and
[`map`,`deepmap`,`flatmap`](https://docs.perl6.org/type/List#routine_map).

You can use Loop phasers together with other Block phasers if you want, but
generally this appears to unnecessary.

Asynchronous phasers
--------------------
These phasers are applicable only inside a
[supply](https://docs.perl6.org/language/concurrency#index-entry-supply_%28on-demand%29)
or [react / whenever](https://docs.perl6.org/language/concurrency#react_and_whenever)
block when doing event driven programming using
[Supplies](https://docs.perl6.org/language/concurrency#Supplies).

| Name | Description |
|:----------|:-----------|
| LAST  | run when a Supply is done |
| QUIT  | run when a Supply terminates with an error |
| CLOSE | run when a Supply is closed |

A deeper review into event driven programming will be left for a future
article in this series.

Summary
=======
