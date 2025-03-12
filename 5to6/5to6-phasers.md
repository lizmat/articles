Perl 6 Phasers 101
==================

This is the sixth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the
[special blocks in Perl 5](https://perldoc.pl/perlmod#BEGIN,-UNITCHECK,-CHECK,-INIT-and-END)
such as `BEGIN` and `END`, and the possibly subtle change in semantics with
so-called [phasers](https://docs.perl6.org/language/phasers) in Perl 6.

Perl 6 has generalized some other features of Perl 5 as phasers, that weren't
covered by special blocks in Perl 5.  And it has added some more phasers that
are not covered by any (standard) functionality of Perl 5 at all.

> One important feature that one should remember about phasers is that
> they are **not** part of the normal flow of execution of a program.
> The runtime executor decides when a phaser is being run depending on
> type of phaser and context.  Therefore all phasers in Perl 6 are spelled
> in uppercase characters to make them stand out better.

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
Modules in Perl 6 are pre-compiled by default, as opposed to Perl 5 which
does not have any pre-compilation of modules or scripts.

> As a user or developer of Perl 6 modules, you do not have to think about
> whether a module should be pre-compiled (again) or not: this is all done
> automatically under the hood when installing a module.  And it is done
> automatically on first usage after an update of Rakudo Perl 6.  And as
> a developer, it is done automaticall every time you have made a change
> to a module you are developing.  The only thing you might notice is a
> small delay when loading a module (again) in such a case.

This means that the `BEGIN` block is only executed whenever the
pre-compilation occurs, **not** every time the module is loaded.  This is
different from Perl 5, where modules ordinarily only exist as source code
that is compiled whenever a module is loaded (even though that module can
load already compiled native library components).

This may cause some unpleasant surprises when porting code from Perl 5 to
Perl 6, because pre-compilation may have happened a long time ago, or even
on a different machine altogether (in case of having installed from an
OS distributed package).  Consider the case of using the value of an
environment variable to enable debugging.  In Perl 5 you could write this as:

    # Perl 5
    my $DEBUG;
    BEGIN { $DEBUG = $ENV{DEBUG} // 0 }

And this would work fine in Perl 5, as the module is compiled every time
it is loaded.  And thus the `BEGIN` block is run every time the module is
loaded.  And the value of `$DEBUG` will be correct, depending on the setting
of environment variable.  But not so in Perl 6.  Because the `BEGIN` phaser
is executed only once when pre-compiling, the `$DEBUG` variable will have
the value that was determined at module pre-compilation time, **not** at
module loading time!

An easy work-around would be to inhibit pre-compilation of a module in Perl 6:

    # Perl 6
    no precompilation;  # this code should not be pre-compiled

However, pre-compilation has several advantages that you do not want to
dismiss that easily:

- Setup of data structures only needs to be done once.  If you have data
structures that you would need to set up each time a module is loaded, you
now do that once when a module is pre-compiled.  Which may be a huge time
/ CPU saver if the module is loaded very often.

- It can load modules **much** faster.  Because it doesn't need to parse
any source code, a pre-compiled module loads much faster than one that needs
to be compiled over and over again.  The prime example of this is the core
setting of Perl 6: the part of Perl 6 that is actually written in Perl 6.
This currently exists of a 64KLOC / 2MB+ source file (generated from many
separate source files for maintainability).  It takes about 1 minute to
compile this source file during installation of Perl 6.  It takes about
125 msecs (at moment of writing this article) to load this pre-compiled
code at Perl 6 startup.  Which is almost a **500x** speedup!

Some other features of Perl 5 **and** Perl 6 that implicitly use `BEGIN`
functionality, have the same caveat.  Take this example where we want to
have a constant `DEBUG` to either have the value of the environment variable
`DEBUG`, or if that is not available, the value `0`:

    # Perl 5
    use constant DEBUG => $ENV{DEBUG} // 0;

    # Perl 6
    my constant DEBUG = %*ENV<DEBUG> // 0; 

The best equivalent in Perl 6 is probably using an `INIT` phaser:

    # Perl 6
    INIT my \DEBUG = %*ENV<DEBUG> // 0;  # sigilless variable bound to value

As in Perl 5, the `INIT` phaser is run just before execution starts.  Of
course, you can use this behaviour of Perl 6 with regards to  module
pre-compilation as a feature:

    # Perl 6
    say "This module was compiled at { BEGIN DateTime.now }";
    # This module was compiled at 2018-10-04T22:18:39.598087+02:00

But more about that syntax later.

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
If you are only interested in finding out how the special blocks of Perl 5
work in Perl 6, you can skip the rest of the article.  Of course, you will
be missing out on quite a few nice features that people have found useful
enough to implement.

Block and Loop phasers
----------------------
Block and Loop phasers are always associated with the surrounding Block,
regardless of where they are located in the block, just the way that the
`CATCH` and `CONTROL` phasers are.  Except you are not limited to only
having one of each: although you could argue that having more than one
doesn't improve maintainability.

> Please note that any `sub` or `method` is *also* considered a Block with
> regards to these phasers.

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

The `LEAVE` phaser corresponds with the `DEFER`functionality in many other
modern programming languages.

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
will be automatically committed by the `KEEP` phaser.

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

If you would only like to check if a Block returns a specific value or type,
you are probably better of specifying a return signature for the Block:

    # Perl 6
    {
        POST { $_ ~~ Int }   # check if the return value is an Int
        ...                  # calculate result
        $result;
    }

is just a very roundabout way of saying:

    # Perl 6
    --> Int {                # return value should be an Int
        ...                  # calculate result
        $result;
    }

So in general, you would only use a `POST` phaser if the necessary checks
would be very involved and reducible to a simple type check.

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
| FIRST | run before the first iteration |
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
[`repeat/while`, `repeat/until`](https://docs.perl6.org/language/control#repeat/while,_repeat/until),
[`for`](https://docs.perl6.org/language/control#for) and
[`map`,`deepmap`,`flatmap`](https://docs.perl6.org/type/List#routine_map).

You can use Loop phasers together with other Block phasers if you want, but
generally this is usually unnecessary.

Summary
=======
Perl 5 has a number of special blocks which basically all have an identical
counterpart in Perl 6.  These special blocks are called "*phasers*" in Perl 6.
Additionally, Perl 6 has a number of special purpose phasers related to blocks
of code and looping constructs, which are described in this article.
Perl 6 also has phasers related to exception handling and warnings,
event driven programming, and document (pod) parsing, but these are *not*
described in this article: these will be described in future articles.
