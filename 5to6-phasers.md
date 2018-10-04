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

| Perl 5     | Perl 6      | Notes      |
|:----------:|:-----------:|:----------:|
| BEGIN      | BEGIN       | not run when loading precompiled code |
| UNITCHECK  | CHECK       | |
| CHECK      |             | no equivalent in Perl 6 |
| INIT       | INIT        | |
| END        | END         | |

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
Perl 6.  An easy work-around would be to inhibit pre-compilation of a module
in Perl 6:

    # Perl 6
    no precompilation;  # this code should not be pre-compiled

However, precompilation has a **very** big advantage: it can load modules
**much** faster because it doesn't need to parse any source code.  The prime
example of this is the core setting of Perl 6: the part of Perl 6 that is
actually written in Perl 6.  This currently exists of a 64KLOC / 2MB+ source
file (generated from many shorter source files for maintainability).  It
takes about 1 minute to compile this source file during installation of
Perl 6.  It takes about 125 msecs to load this pre-compiled code at Perl 6
startup.  Which is almost a **500x** speedup!

Some other features of Perl 5 **and** Perl 6 that implicitely use `BEGIN`
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

INIT
====
The functionality of the [`INIT`](https://docs.perl6.org/language/phasers#INIT)
phaser in Perl 6 is exactly the same as the `INIT` special block in Perl 5.
It specifies a piece of code to be executed **just before** the code in the
compilation unit will be executed.  In pre-compiled modules in Perl 6, it can
serve as an alternative to the `BEGIN` phaser.

END
===
The functionality of the [`END`](https://docs.perl6.org/language/phasers#END)
phaser in Perl 6 is exactly the same as the `END` special block in Perl 5.
It specifies a piece of code to be executed **after** all the code in the
compilation unit has been executed, or the code decides to exit (either
intended, or unintended because of an exception having been thrown).

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
    BEGIN { $foo = %*ENV<FOO> // 42 }

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

This will ensue that the value will be determined at the moment the module
is **loaded**, rather then whenever it was precompiled (which typically
only happens once during installation of the module).

Additional phasers in Perl 6
============================
Perl 6 has generalized some other features of Perl 5 as phasers, that weren't
covered by special blocks in Perl 5.  And it has added some more phasers that
are not covered by any (standard) functionality of Perl 5 at all.

Exception handling phasers
--------------------------
| Name | Description |
|:----------|:-----------|
| CATCH     | run when an exception is thrown |
| CONTROL   | run for any (other) control exception |

Block phasers
-------------
| Name | Description |
|:----------|:-----------|
| ENTER     | run everytime when entering a Block |
| LEAVE     | run everytime when leaving a Block |
| KEEP      | run everytime a Block is left successfully |
| UNDO      | run everytime a Block is left **un**successfully |
| PRE       | check condition *before* running a Block |
| POST      | check return value *after* having run a Block |

ENTER
*****

Loop phasers
------------
| Name | Description |
|:----------|:-----------|
| FIRST     | run only the first time through a loop |
| NEXT      | run after each completed iteration, or with next |
| LAST      | run after the last iteration, or with last |

Asynchronous phasers
--------------------
These phasers are applicable only inside a
[supply](https://docs.perl6.org/language/concurrency#index-entry-supply_%28on-demand%29)
or [react / whenever](https://docs.perl6.org/language/concurrency#react_and_whenever)
block when doing event driven programming using
[Supplies](https://docs.perl6.org/language/concurrency#Supplies).

| Name | Description |
|:----------|:-----------|
| LAST      | run when a Supply is done |
| QUIT      | run when a Supply terminates with an error |
| CLOSE     | run when a Supply is closed |

A deeper review into event driven programming will be left for a future
article in this series.

Summary
=======
