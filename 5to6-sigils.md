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
| **$**  | Scala      | Item        |
| **@**  | Array      | Positional  |
| **%**  | Hash       | Associative |
| **&**  | Subroutine | Callable    |
| **\*** | Glob       | n/a         |

Summary
-------

