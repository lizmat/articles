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
| **$**  | Scalar     | Item        |
| **@**  | Array      | Positional  |
| **%**  | Hash       | Associative |
| **&**  | Subroutine | Callable    |
|   *    | Typeglob   | n/a         |

Typeglobs
=========
As you may have noticed, Perl 6 does not have a ** * ** sigil.  Perl 6 does
**not** have the concept of "typeglobs".  If you don't know what typeglobs
are, then you don't have to worry about this at all: you can get by in Perl 5
very well without having to know the intricacies of the implementation of
symbol tables in Perl 5 (and you can skip the next paragraph).

If you *do* know about typeglobs, one should realize that in Perl 6 the sigil
is part of the name that is stored in a
[symbol table](https://en.wikipedia.org/wiki/Symbol_table). Whereas in Perl 5
the name is stored *without* sigil, referencing an array in which the sigil
serves as an index to the information needed.  For example, in Perl 5, if youi
reference **$foo** in your program, the compiler will look up **"foo"**
(without sigil), and then fetch the associated information (which is an array),
and look up what it needs at the index for the **$** sigil.  In Perl 6, if you
reference **$foo**, the compiler will look up **"$foo"** and directly use the
information associated with that key.

Please do not confuse the ** * ** used in Perl 6 to indicate slurpiness of
parameters, with the typeglob sigil.  If anything, you could consider the
** * ** in that context in Perl 6 as a sort of "pregil", something that
prefixes the sigil.

Summary
-------

