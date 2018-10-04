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
the `BEGIN` phaser in Perl 6: it specifies a piece of code to be executed
**immediately** as soon as it has been parsed (so *before* the program as a
whole has been parsed).

Caveats when using BEGIN in modules in Perl 6
---------------------------------------------

UNITCHECK
=========
The functionality of the `UNITCHECK` block in Perl 5 is performed by the
`CHECK` phaser in Perl 6.  Otherwise, it is exactly the same: it specifies
a piece of code to be executed when *compilation* of the current compilation
unit is done.

CHECK
=====
There is **no** equivalent of the `CHECK` special block in Perl 6.  The main
reason for this is that you probably shouldn't be using the `CHECK` special
block in Perl 5 anymore, but use `UNITCHECK` instead, as its semantics are
much more sane.

INIT
====

END
===

More than special blocks
========================
Phasers in Perl 6 actually have some additional features that make them so
much more than just special blocks.

No need for a block
-------------------
Most phasers in Perl 6 do not have to be a Block (aka, followed by code
between curly braces), but they can also consist of a single statement
*without* any curly braces.  This means that if you've written in Perl 5: 

    # Perl 5
    # need to define lexical outside of BEGIN scope,
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
code / expression.  So you can actually use them in an expression.  So the
above example using `BEGIN` can also be written as:

    # Perl 6
    my $foo = BEGIN %*ENV<FOO> // 42;

When used like that with a `BEGIN` phaser, you are in fact creating a
nameless constant and assigning that at runtime.

Additional phasers in Perl 6
============================

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
