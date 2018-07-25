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

Argument passing in Perl 5
--------------------------
All arguments that you pass to a Perl 5 subroutine are flattened and put into
the automatically defined `@_` array variable inside.  That is basically all
that Perl 5 does with passing arguments to subroutines.  Nothing more, nothing
less.  There are however several idioms in Perl 5 that take it from there.
The most common, I would say "standard" idiom, in my experience is:

    # Perl 5
    sub do_something {
        my ($foo, $bar) = @_;
        # actually do something with $foo and $bar
    }

It basically performs a list assignment (copy) to two (new) lexical variables.
This way of accessing the arguments to a subroutine, is also supported in
Perl 6, but is only intended as a means to make migrations easier.

If you are expecting a fixed number of arguments, followed by a variable
number of arguments, then the following idiom is usually used:

    # Perl 5
    sub do_something {
        my $foo = shift;
        my $bar = shift;
        for (@_) {
            # do something for each element in @_
        }
    }

This idiom depends on the magic behaviour of
[`shift`](https://perldoc.perl.org/functions/shift.html), which shifts from
`@_` in this context.  If the subroutine is intended to be called as a method,
one often sees something like:

    # Perl 5
    sub do_something {
        my $self = shift;
        # do something with $self
    }

as the first argument passed is the invocant in Perl 5.

By the way, this idiom can actually also be written in the first idiom:

    # Perl 5
    sub do_something {
        my ($foo, $bar, @rest) = @_;
        for (@rest) {
            # do something for each element in @rest
        }
    }

But that would be less efficient, as it would involve copying of a potentially
long list of values.

The third idiom revolves about directly accessing the `@_` array.

    # Perl 5
    sub sum_two {
        return $_[0] + $_[1];  # return the sum of the two parameters
    }

This idiom is usually only used for small, usually one-liner subroutines,
as this is one of the most efficient ways of handling arguments, as no
copying whatsoever is taking place.

This idiom is also used if one wants to change any variable that is passed
as a parameter.  Since the elements in `@_` are aliases to any variables
specified (in Perl 6 one would say: "are bound to the variables"), it is
possible to change the contents:

    # Perl 5
    sub make42 {
        $_[0] = 42;
    }
    my $a = 666;
    make42($a);
    say $a;      # 42

Named arguments in Perl 5
-------------------------
Named arguments as such **do not exist** in Perl 5.  There is however an often
used idiom that effectively mimicks named arguments:

    # Perl 5
    sub do_something {
        my %named = @_;
        if (exists %named{bar}) {
            # do stuff if named variable "bar" exists
        }
    }

This basically initializes the hash `%named` by alternately taking a key
and a value from the `@_` array.  If you call a subroutine with arguments
using the fat-comma syntax:

    # Perl 5
    frobnicate( bar => 42 );

it will in fact pass 2 values, `"foo"` and `42`, which will then be placed
into the `%named` hash as the value `42` associated with key `"foo"`.  But
the same would have happened if you had specified:

    # Perl 5
    frobnicate( "bar", 42 );

The `=>` is syntactic sugar for automatically quoting the left hand side.
Otherwise, it functions just like a comma (hence the name "fat comma").

If a subroutine is called as a method with named arguments, this idiom gets
combined with the standard idiom:

    # Perl 5
    sub do_something {
        my ($self, %named) = @_;
        # do something with $self and %named
    }

or, alternately:

    # Perl 5
    sub do_something {
        my $self  = shift;
        my %named = @_;
        # do something with $self and %named
    }

Argument Passing in Perl 6
--------------------------
In their simplest form, subroutine signatures in Perl 6 are very much like
the "standard" idiom of Perl 5.  But instead of being part of the code,
they are part of the definition of the subroutine, and you don't need to
do the assignment:

    # Perl 6
    sub do-something($foo, $bar) {
        # actually do something with $foo and $bar
    }

versus:

    # Perl 5
    sub do_something {
        my ($foo, $bar) = @_;
        # actually do something with $foo and $bar
    }

In Perl 6, the `($foo, $bar)` part is called the *signature* of the subroutine.

Since Perl 6 has an actual `method` keyword, it is not necessary to
take the invocant into account, as that is automatically available with the
`self` term:

    # Perl 6
    class Foo {
        method do-something-else($foo, $bar) {
            # do something else with self, $foo and $bar
        }
    }

Such parameters are called *positional parameters* in Perl 6.  Unless
indicated otherwise, positional parameters **must** be specified when
calling the subroutine.

If you need the aliasing behaviour of using `$_[0]` directly in Perl 5, you
can mark the parameter as writable by specifying the `is rw` trait:

    # Perl 6
    sub make42($foo is rw) {
        $foo = 42;
    }
    my $a = 666;
    make42($a);
    say $a;      # 42

When you pass an array as an argument to a subroutine, it does **not** get
flattened in Perl 6.  The only thing you need to do, is to accept an array
as an array in the signature:

    # Perl 6
    sub handle-array(@a) {
        # do something with @a
    }
    my @foo = "a" .. "z";
    handle-array(@foo);

You can pass any number of arrays:

    # Perl 6
    sub handle-two-arrays(@a, @b) {
        # do something with @a and @b
    }
    my @bar = 1..26;
    handle-two-arrays(@foo, @bar);

If you **do** want the
([variadic](https://en.wikipedia.org/wiki/Variadic_function)) flattening
semantics of Perl 5, then you can indicate this with a so-called "slurpy"
array.  This indicated by prefixing the array with an asterisk in the
signature:

    # Perl 6
    sub slurp-an-array(*@values) {
        # do something with @values
    }
    slurp-an-array("foo", 42, "baz");

A slurpy array can only occur as the last positional parameter in a signature.

If you prefer to use the Perl 5 way of specifying parameters in Perl 6, you
can do this by specifying a slurpy array `*@_` in the signature:

    # Perl 6
    sub do-like-5(*@_) {
        my ($foo, $bar) = @_;
    }

Named arguments in Perl 6
-------------------------
On the calling side, named arguments in Perl 6 can be expressed in a way that
is very similar to the way they are expressed in Perl 5:

    # Perl 5 and Perl 6
    frobnicate( bar => 42 );

However, on the side of the definition of the subroutine, things are very
different:

    # Perl 6
    sub frobnicate(:$bar) {
        # do something with $bar
    }

The difference between an ordinary (positional) Parameter and a named
Parameter, is the colon, which precedes the sigil and the variable name in
the definition:

    $foo      # positional parameter, receives in $foo
    :$bar     # named parameter "bar", receives in $bar

Unless otherwise specified, named parameters are *optional*.  If a named
argument is not specified, then the associated variable will contain the
default value, which usually is the type object `Any`.

If want to catch *any* (other) named arguments, you can use a so-called
"slurpy hash".  Just like the "slurpy array", this is indicated by an
asterisk prefixing a hash:

    # Perl 6
    sub slurp-nameds(*%nameds) {
        say "Received: " ~ join ", ", sort keys %nameds;
    }
    slurp-nameds(foo => 42, bar => 666); # Received: bar, foo

As with the slurpy array, there can only be one slurpy hash in a signature,
and it needs to be specified after any other named parameters.

> Oftentimes when you want to pass a named argument to a subroutine from a
> variable with the same name.  In Perl 5 this looks like:
> `do_something(bar => $bar)`.  In Perl 6, you can specify this in the same
> way: `do-something(bar => $bar)`.  But you can also use a shortcut:
> `do-something(:$bar)`.  This means less typing, and less chance of making
> typos.

Default values in Perl 6
------------------------
Perl 5 has the following idiom for making parameters optional with a default
value:

    # Perl 5
    sub dosomething_with_defaults {
        my $foo = @_ ? shift : 42;
        my $bar = @_ ? shift : 666;
        # actually do something with $foo and $bar
    }

In Perl 6, you can specify default values as part of the signature by
specifying an equal sign and an expression:

    # Perl 6
    sub dosomething-with-defaults($foo = 42, :$bar = 666) {
        # actually do something with $foo and $bar
    }

Positional parameters become optional if they have a default value specified
for them.  Named parameters stay optional regardless of any default value.

> This has touched on the Perl 6 features that you would like to be familiar
> with when coming from Perl 5.  Signatures in Perl 6 have many more
> interesting features.  If you want to know more about these features, you
> can check out the
> [documentation of the Signature object](https://docs.perl6.org/type/Signature).

Summary
-------
Perl 6 has a way of describing how arguments to a subroutine should be
captured into parameters of that subroutine.  Positional parameters are
indicated by their name and the appropriate sigil (e.g. `$foo`).  Named
parameters are prefixed with a colon (e.g. `:$bar`).  Positional parameters
can be marked as `is rw` to allow changing variables in the callers scope.

Positional arguments can be flattened in a slurpy array, which is prefixed
by an asterisk (e.g. `*@values`).  Unexpected named arguments can be
collected using a slurpy hash, which is also prefixed with an asterisk
(e.g. `*%nameds`).

Default values can be specified inside the signature by adding an expression
after an equal sign (e.g. `$foo = 42`), which makes that parameter optional.
