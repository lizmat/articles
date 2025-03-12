# Not So Obvious Semantic Changes (Part 2 of 3)

## About $_
In Perl, `$_` is a global variable which may be localised for certain scopes, like in a `for` loop.  This means that it is trivial in Perl to make a subroutine that accesses `$_` of the caller's context:
```
# Perl
sub topic_squared {
    say $_ * $_;
}
$_ = 42;
topic_squared;  # 1764
```
However, in Raku, `$_` (aka “ the topic variable “) is **always** lexical.  This means that it is (almost) impossible to write a subroutine that uses the value of `$_` from the scope of the caller.

This also means that idiom such as a bare `say` cannot work in Raku, as basically almost all commands in Raku are just subroutines.  So it was decided to not support that idiom in Raku:
```
# Raku
for 1..10 {
    say;  # Unsupported use of bare “say"
}
```
Fortunately, there **is** a syntax that is almost as small in Raku:
```
# Raku
for 1..10 {
    .say;  # calls method "say" on $_
}
```
This calls the `say` method on `$_`.  As we’ve seen, the period is used in Raku to indicate method calling.  If you call a method on "nothing", then it will assume the current topic, so will use `$_` for that.

## $a and $b are not special
In Perl, the variables `$a` and `$b` are special in that you do not have to define them before you can use them:
```
# Perl
use strict;  # MUST define variables before using them
$a = 42;     # except $a
$b = 666;    # and $b

say $a;  # 42
say $b;  # 666
```
So why does Perl have that?  Well, Perl will fill those variables in certain contexts for you, specifically when doing a `sort`:
```
# Perl
my @sorted = sort {
    $a cmp $b    # $a and $b set to each value in turn to compare
} @files;
```
In Raku there are several ways to do the same, but the easiest to remember is the auto-signature syntax:
```
# Raku
my @sorted = sort {
    $^a cmp $^b    # generates signature -> $a, $b
}, @files;
#↑ note: comma is needed
```
Note that the `^` in this example is called a ["twigil"](https://docs.raku.org/language/variables#The_^_twigil) (as in "secondary sigil").

More on signatures in future blog posts.

# Summary
In this blog post the subtle different semantics of `$_` between Perl and Raku are handled, and it is shown that Raku doesn't have the direct equivalent of Perl's `$a` and `$b`, but that there is a more flexible alternative using secondary sigils.
