# Obvious Syntactic Changes between Perl and Raku (Part 1 of 3)
Even though Perl and Raku share the same ancestry, they differ in a number of very obvious syntactic ways.  This blog post highlights the ones that I found to be causing most of the problems with people coming from Perl.  This chapter also uses some concepts and syntax that will be further explained in following chapters.  So please, if there is something that you don’t understand here, just leave it be for now.  It will all become clear soon enough!

## Use Strict
In Raku, the equivalent of `use strict` is always active, which in Perl is not the case by default:

    # Perl
    $a = 42;
    say $a;   # 42

Compare to Raku:
    # Raku
    $a = 42;
    say $a;
    # ===SORRY!=== Error while compiling …
    # Variable '$a' is not declared
    # at -e: … # ———> <BOL>⏏$a = 42

If you really want to be lax in Raku like in Perl, then you can actually remove the strictness with `no strict`:

    # Raku
    no strict;
    $a = 42;
    say $a;  # 42

But this is usually a bad idea.

## Use Warnings
In Raku, the equivalent of `use warnings` is always active, which in Perl is not the case by default:

    # Perl
    use warnings;  # must activate warnings
    my $a;
    say $a + 1;
    # Use of uninitialized value $a in addition (+) at …
    # 1

Versus in Raku:

    # Raku
    my $a;
    say $a + 1;
    # Use of uninitialized value of type Any in numeric context
    #   in … at … line …
    # 1

Disabling warnings in a section is achieved by using a `quietly` block:

    # Raku
    my $a;
    quietly {        # silence any warnings inside this block
        say $a + 1;
    }
    # 1

More on this in the blog post about phasers.

## Identifiers
In Raku, you may use hyphens as part of [identifiers](https://docs.raku.org/syntax/identifiers#Ordinary_identifiers) as long as they do not start the identifier, and are followed by an alphabetic character:

    # Perl
    my $the-answer = 42;  # syntax error

    # Raku
    my $the-answer = 42:  # just fine

Using hyphens in identifiers, is sometimes referred to as using "kebab-case".

Also note that all characters that are considered alphabetic / alphanumeric according to the Unicode standard, are allowed in Raku:

    # Raku
    my $駱駝道 = 42;  # the way of the camel

Of course, in Raku you can still use the underscore in identifiers like you would in Perl.

    # Raku
    my $the_answer = 42;  # also works

## String Concatenation Vs Calling Methods
String concatenation in Raku uses the tilde `~` for concatenating strings.  In Perl the period is used for that:

    say "The quick brown fox " . $action;  # Perl
    say "The quick brown fox " ~ $action;  # Raku

More generally, the tilde `~` indicates “stringy” things in Raku.

Calling a method on an object in Raku is indicated with a period, followed by the name of the method, a syntax that many other programming languages also use.  In Perl, the `->` characters (minus, greater than) are used:

    #      ↓↓
    $object->frobnicate;  # Perl
    $object.frobnicate;   # Raku
    #      ↑

## (Lack Of) Whitespace
Whitespace is slightly more important in Raku than it is in Perl.  On the other hand, Raku needs a lot less parentheses than Perl.  Where it is legal to say in Perl:

    # Perl
    if($result == 42) {  # note missing space between “if” and (
        say “The answer”;
    }

In Raku, an item directly followed by a parenthesis open is always interpreted as calling that item with the given parameters between parentheses.  So this would be interpreted as an attempt to call the `if` subroutine with the result of the comparison `$result == 42`.  But since you most likely do not have a subroutine named `if`, this will result in a compilation error in Raku.

So the proper way to do this in Raku is to make sure there is whitespace between the `if` and the condition:
    # Perl and Raku
    if ($result == 42) {  # note space between “if” and (
        say “The answer”;
    }

However, you do not need to use parentheses in many situations in Raku where you *would* need to use them in Perl.  This is one such an example:

    # Raku
    if $result == 42 {    # note absence of parentheses
        say “The answer”;
    }

Another example are `for` loops:

    # Raku
    for @items {
        # do something
    }

Of course, this also implies that if you’re used to having whitespace between the name of a subroutine and its parameters in Perl:

    # Perl
    sub sum_two { my ($a,$b) = @_; $a + $b }
    
    say sum_two (42,666);  # 708

That doesn’t work in Raku like you think it would:

    # Raku
    sub sum-two ($a, $b) { $a + $b }

    say sum-two(42,666);   # 708
    say sum-two (42,666);  # Too few positionals passed; expected 2 arguments but got 1

What happens here is that in Raku `(42,666)` is a list with two values.  But it acts as a single argument in the call to the subroutine because of the whitespace between the name of the sub and the arguments.  So always keep the parenthesis open cuddled to the name of the subroutine / method in Raku.

If you have a style sensibility regarding this, you can actually use a `slang` (namely the ecosystem module [`Slang::Tuxic`](https://raku.land/github:FROGGS/Slang::Tuxic)) to change this syntactic property in your code:

    # Raku
    use Slang::Tuxic;

    sub sum-two ($a, $b) { $a + $b }

    say sum-two(42,666);    # 708
    say sum-two (42,666);   # 708

Unfortunately, this also means that you will have to specify parentheses like in Perl with structures such as `if`, `for`, etc.

## Summary

In this blog post it was shown that some sane default boilerplate in Perl is default in Raku (`use strict`, `use warning`, that you can use hyphens in identifiers (aka "kebab-case"), that the period (`.`) is used for calling methods, and the tilde (`~`) is used for concatenation.

It was also shown that you can use way fewer parentheses in Raku, at the expense of having slightly stricter whitespace rules.
