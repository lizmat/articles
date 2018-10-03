Phasers 101
===========

This is the sixth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the
[special blocks in Perl 5](https://perldoc.pl/perlmod#BEGIN,-UNITCHECK,-CHECK,-INIT-and-END)
such as `BEGIN` and `END`, and the possible subtle change in semantics with
so-called [phasers](https://docs.perl6.org/language/phasers) in Perl 6.

An overview
===========
Let's start with an overview again of Perl 5 special blocks and their Perl 6
counterparts.

| Perl 5     | Perl 6      | Notes      |
|:----------:|:-----------:|:----------:|
| BEGIN      | BEGIN       | not run when loading precompiled code |
| UNITCHECK  | CHECK       | |
| CHECK      |             | no equivalent in Perl 6 |
| INIT       | INIT        | |
| END        | END         | |

Summary
-------
