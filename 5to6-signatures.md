Signatures in Perl 6
====================

The third installment of the
[Migrating Perl 5 code to Perl 6](5to6-introduction.md) series.  The first
installment was about
[garbage collection and how timely destruction works](5to6-finalizing.md),
the second was about
[how references were replaced by the use of containers](5to6-containers.md).

In this post I will be focussing on (subroutine) signatures in Perl 6
and how they differ from the situation in Perl 5.

Experimental signatures in Perl 5
---------------------------------
If you're migrating Perl 5 code to Perl 6, then most likely you are not
using the
[experimental signature feature](https://metacpan.org/pod/distribution/perl/pod/perlsub.pod#Signatures)
that has become available in Perl 5 since 5.20, or any of the longer existing
CPAN modules like [signatures](https://metacpan.org/pod/signatures),
[Function::Parameters](https://metacpan.org/pod/Function::Parameters) or
any of the other Perl 5 modules on CPAN that
[have "signature" in their name](https://metacpan.org/search?q=signature).

Also, in my experience,
[prototypes](https://metacpan.org/pod/perlsub#Prototypes) generally have had
little usage in most Perl programs out in the world (aka
[DarkPan](http://modernperlbooks.com/mt/2009/02/the-darkpan-dependency-management-and-support-problem.html)).
I will therefore only compare the Perl 6 functionality with the most common
usage of "classic" Perl 5 argument passing.

Argument passing in Perl 5
--------------------------
All arguments that you pass to a Perl 5 subroutine are flattened and put into
the `@_` array inside.  That is basically all that Perl 5 does with passing
arguments to subroutines.  Nothing more, nothing less.  There are however
several idioms in Perl 5 that take it from there.  The most common, I would
say "standard" idiom, in my experience is:

    my ($foo, $bar) = @_;

It basically performs a list assignment (copy) to two (new) lexical variables.
This way of accessing the arguments to a subroutine, is also supported in
Perl 6, but is only intended as a means to make migrations easier.

If you are expecting a fixed number of arguments, followed by a variable
number of arguments, then the following idiom is usually used:

    my $foo = shift;
    my $bar = shift;
    for (@_) {
        # do something for each element in @_
    }

This idiom depends on the magic behaviour of
[`shift`](https://perldoc.perl.org/functions/shift.html), which shifts from
`@_` in this context.  If the subroutine is intended to be called as a method,
one often sees something like:

    my $self = shift;

as the first argument passed is the invocant in Perl 5.

By the way, this idiom can actually also be written in the first idiom:

    my ($foo, $bar, @rest) = @_;
    for (@rest) {
        # do something for each element in @rest
    }

But that would be less efficient, as it would involve copying of a potentially
long list of values.

The third idiom revolves about directly accessing the `@_` array.

    return $_[0] + $_[1];  # return the sum of the two parameters

This idiom is usually only used for small, usually one-liner subroutines,
as this is one of the most efficient ways of handling arguments, as no
copying whatsoever is taking place.

This idiom is also used if one wants to change any variable that is passed
as a parameter.  Since the elements in `@_` are aliases to any variables
specified (in Perl 6 one would say: "are bound to the any variables"), it
is possible to change the contents:

    sub make42 { $_[0] = 42 }
    my $a = 666;
    make42($a);
    say $a;      # 42

Named arguments in Perl 5
-------------------------
Named arguments as such do not exist in Perl 5.  There is however an often
used idiom that effectively mimicks named arguments:

    my %named = @_;
    if (exists %named{foo}) {   # named variable 
        # do stuff if named variable "foo" exists
    }

This basically initializes the hash `%named` by alternately taking a key
and a value from the `@_` array.  If you call a subroutine with arguments
using the fat-comma syntax:

    frobnicate( foo => 42 );

it will in fact pass 2 values, `"foo"` and `42`, which will then be placed
into the `%named` hash as the value `42` associated with key `"foo"`.  But
the same would have happened if you had specified:

    frobnicate( "foo", 42 );

The `=>` is syntactic sugar for automatically quoting the left hand side.
Otherwise, it functions just like a comma (hence the name "fat comma").

If a subroutine is called as a method with named arguments, this idiom gets
combined with the standard idiom:

    my ($self, %named) = @_;

or, alternately:

    my $self = shift;
    my %named = @_;

Signatures in Perl 6
--------------------
Subroutines signatures in Perl 6 in their simplest form

Summary
-------
Perl 6 has a way of describing how parameters should be taken and/or coerced,
and allows for subroutines (and methods) to have the same name but a different
set of acceptable parameters (Signature).  The value of the arguments is used
by the Perl 6 runtime to select the subroutine (method) to actually call
([multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch)).
