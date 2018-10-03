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



UNITCHECK
=========

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
Most phasers in Perl 6 do not have to be a Block, but they can also consist
of a single statement *without* any curly braces.  This means that if you've
written in Perl 5: 

    # Perl 5
    my $foo;             # need to define lexical outside of BEGIN scope,
    BEGIN { $foo = 42 }  # otherwise it won't be known in the rest of the code

you can write this in Perl 6 as:

    # Perl 6
    BEGIN my $foo = 42;  # share scope with surrounding code


Block phasers
=============
| Name | Description      |
|:----------|:-----------|
| ENTER     | run everytime when entering a Block |
| LEAVE     | run everytime when leaving a Block |
| KEEP      | run everytime a Block is left successfully |
| UNDO      | run everytime a Block is left **un**successfully |
| PRE       | check condition *before* running a Block |
| POST      | check return value *after* having run a Block |

Loop phasers
============
| Name | Description      |
|:----------|:-----------|
| FIRST     | run only the first time through a loop |
| NEXT      | run after each completed iteration, or with next |
| LAST      | run after the last iteration, or with last |

Exception handling phasers
==========================
| Name | Description      |
|:----------|:-----------|
| CATCH     | run when an exception is thrown |
| CONTROL   | run for any (other) control exception |

Asynchronous phasers
====================
| Name | Description      |
|:----------|:-----------|
| LAST      | run  |
| QUIT      | run  |
| CLOSE     | run  |

Summary
=======
