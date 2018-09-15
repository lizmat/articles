Sigils in Perl 6
================

In the [first article](5to6-introduction.md) in this series comparing Perl 5
to Perl 6, we looked into some of the issues you might encounter when migrating
code into Perl 6. In the [second article](5to6-finalizing.md), we examined how
garbage collection works in Perl 6. In the [third article](5to6-containers.md),
we looked at how containers replaced references in Perl 6, and in the
[fourth article](5to6-signatures.md), we focused on (subroutine) signatures
in Perl 6 and how they differ from those in Perl 5.

Here, in the fifth article, we will look at the subtle differences of the use
of [sigils](https://www.perl.com/article/on-sigils/) between Perl 5 and Perl 6.

An overview
===========
Let's start with an overview of what sigils are associated with:

| Sigil  | Perl 5     | Perl 6      |
|:------:|:----------:|:-----------:|
| **@**  | Array      | Positional  |
| **%**  | Hash       | Associative |
| **&**  | Subroutine | Callable    |
| **$**  | Scalar     | Item        |
|   *    | Typeglob   | n/a         |

@ - Array vs Positional
=======================
When you define an array in Perl 5, you basically create an expandable list of
scalar values and give it a name with the sigil **@**:

    # Perl 5
    my @foo = (1,2,3);
    push @foo, 42;
    say for @foo;  # 1␤2␤3␤42␤

When you define an array like that in Perl 6, you basically create a new
[Array](https://docs.perl6.org/type/Array) object and **bind** that to the
entry by that name in the lexical pad.  So:

    # Perl 6
    my @foo = 1,2,3;

is in fact syntactic sugar:

    # Perl 6
    my @foo := Array.new( 1,2,3 );

The **@** sigil in Perl 6 indicates a type constraint: if you want to bind
something into a lexpad entry with that sigil, it must perform the
[Positional](https://docs.perl6.org/type/Positional.html) role.  One can
easily introspect whether a class performs a certain role using smartmatch:

    # Perl 6
    say Array ~~ Positional;   # True

So one could argue that all arrays in Perl 6 are implemented in the
equivalent of a [tied array](https://perldoc.perl.org/functions/tie.html)
in Perl 5.  And that would not be far from the truth.  Just like you can
create a class to *tie* with that is a subclass from a class providing basic
functionality in Perl 5, in Perl 6 you use
[role composition](https://docs.perl6.org/language/objects#Roles) with a
role that provides basic functionality, so that you only need to implement
some specific functionality appropriate to your use case.

Without going too deep into the specifics, a simple example might elucidate.
One of the key methods that a class implementing the *Positional* role should
implement, is the `AT-POS` method.  This method gets called whenever a
single element needs to be accessed.  So, when you write:

    say @a[42];

you are in fact executing:

    say @a.AT-POS(42);

Of course, this is not the only method that you could implement, there
are [many more](https://docs.perAl6.org/language/subscripts#Methods_to_implement_for_positional_subscripting).

Rather than having to *bind* your class performing the *Positional* role,
there's a special syntax using the *is* trait.  So instead of having to
write:

    # Perl 6
    my @a := YourClass.new( 1,2,3 );

one can write:

    # Perl 6
    my @a is YourClass = 1,2,3;

In Perl 5, *tied* arrays are notoriously slow.  In Perl 6, arrays are
similarly slow at startup.  Fortunately, Rakudo Perl 6 optimizes hot code
paths by inlining and JITting opcodes to machine code where possible (and
with recent advancements in the optimizer, this happens more, sooner and
better).

% - Hash vs Associative
=======================
Hashes in Perl 6 are implemented in a similar way to arrays: you could also
consider them a *tied* hash, using Perl 5 terminology.  Instead of the
*Positional* role that is used to implement arrays, the
[Associative](https://docs.perl6.org/type/Associative) role should be used
to implement hashes.

Again, a simple example might elucidate.  One of the key methods that a class
implementing the *Associative* role should implement, is the `AT-KEY` method.
This method gets called whenever the value of a specific key needs to be
accessed.  So, when you write:

    say %h<foo>;

you are in fact executing:

    say %h.AT-KEY("foo");

Of course there are
[many more](https://docs.perAl6.org/language/subscripts#Methods_to_implement_for_associative_subscripting) methods that you could implement.

% - Subroutine vs Callable
==========================

$ - Scalar vs Item
==================



\* - Typeglobs
==============
As you may have noticed, Perl 6 does not have a * sigil.  Perl 6 does
**not** have the concept of "typeglobs".  If you don't know what typeglobs
are, then you don't have to worry about this at all: you can get by in Perl 5
very well without having to know the intricacies of the implementation of
symbol tables in Perl 5 (and you can skip the next paragraph).

> If you *do* know about typeglobs, one should realize that in Perl 6 the sigil
> is part of the name that is stored in a
> [symbol table](https://en.wikipedia.org/wiki/Symbol_table), whereas in Perl 5
> the name is stored *without* sigil.  For example, in Perl 5, if you
> reference **$foo** in your program, the compiler will look up **"foo"**
> (without sigil), and then fetch the associated information (which is an
> array), and look up what it needs at the index for the **$** sigil.  In
> Perl 6, if you reference **$foo**, the compiler will look up **"$foo"**
> and directly use the information associated with that key.

Please do not confuse the * used in Perl 6 to indicate slurpiness of
parameters, with the typeglob sigil.  If anything, you could consider the *
in that context as a sort of "pregil", something that *prefixes* a sigil.

Summary
-------

