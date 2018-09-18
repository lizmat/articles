Sigils in Perl 5 and Perl 6
===========================

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
    push @foo, 42;
    .say for @foo;  # 1␤2␤3␤42␤

is functionally the same as in Perl 5.  However, the first line is in fact
syntactic sugar for:

    # Perl 6
    my @foo := Array.new( 1,2,3 );

This **binds** (rather than assigns) a new `Array` object to the lexically
defined name `@foo`.  The **@** sigil in Perl 6 indicates a type constraint:
if you want to bind something into a lexpad entry with that sigil, it **must**
perform the [Positional](https://docs.perl6.org/type/Positional.html) role.
One can easily introspect whether a class performs a certain role using
smartmatch:

    # Perl 6
    say Array ~~ Positional;   # True

One could argue that all arrays in Perl 6 are implemented in the equivalent
of a [tied array in Perl 5](https://perldoc.perl.org/functions/tie.html).
And that would not be far from the truth.  Just like you can create a class
to *tie* with that is a subclass from a class providing basic functionality
in Perl 5, in Perl 6 you use
[role composition](https://docs.perl6.org/language/objects#Roles) with a
role that provides basic functionality, so that you only need to implement
some specific functionality appropriate to your use case.  There's even a
a Perl 6 equivalent for Perl 5's
[Tie::Array](https://metacpan.org/pod/Tie::Array) called
[Array::Agnostic](https://modules.perl6.org/dist/Array::Agnostic).

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

In Perl 5, *tied* arrays are notoriously slow compared to "normal" arrays.
In Perl 6, arrays are similarly slow at startup.  Fortunately, Rakudo Perl 6
optimizes hot code paths by inlining and JITting opcodes to machine code
where possible (and with recent advancements in the optimizer, this happens
more, sooner and better).

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

& - Subroutine vs Callable
==========================
In Perl 5 there is only type of callable executable code, the subroutine:

    # Perl 5
    sub frobnicate { shift ** 2 }

And if you want to pass on a subroutine as a parameter, you need to get a
reference to it:

    # Perl 5
    sub do_stuff_with {
        my $lambda = shift;
        &$lambda(shift);
    }
    say do_stuff_with( \&frobnicate, 42 );  # 1764

In Perl 6 there are multiple types of objects that can contain executable
code.  What they have in common, is that they consume the
[Callable role](https://docs.perl6.org/type/Callable).

The `&` sigil forces binding to an object performing the `Callable` role,
just like the `%` sigil does with the `Associative` role, and the `@` sigil
does with the `Positional` role.  An example very close to Perl 5:

    # Perl 6
    my &foo = sub ($a,$b) { $a + $b }
    say foo(42,666);  # 708

Note that even though the variable has the `&` sigil, you do **not** need
to use that when you want to execute the code in that variable.  In fact,
if you would run the code in a `BEGIN` block, there would be no difference
at all with an ordinary `sub` declaration:

    # Perl 6
    BEGIN my &foo = sub ($a,$b) { $a + $b }

Note that, contrary to Perl 5, `BEGIN` blocks in Perl 6 can be a single
statement **without** a block, so that it shares its lexical scope with
the outside.

The main advantage to using a `&` sigilled variable, is that it is known
at compile time that there is going to be something executable in there,
even it is not known yet what at that time.

There are other ways to set up a piece of code for execution:

    # Perl 6
    my &goo = -> $a, $b { $a + $b }  # same, using a Block with a signature
    my &hoo = { $^a + $^b }          # same, using auto-generated signature
    my &ioo = * + *;                 # same, using Whatever currying

If you'd like to know more:

- [Block with a signature](https://docs.perl6.org/type/Block)
- [Autogenerated signatures](https://docs.perl6.org/language/variables#index-entry-%24%5E)
- [Whatever currying](https://docs.perl6.org/type/Whatever)

Which one you may or can use, really depends on the situation, and your
personal preferences.

Finally, you can also use the `&` sigil inside a signature to indicate that
the callee wants something executable there.  Which brings us back to the
first two code examples in this section:

    # Perl 6
    sub frobnicate { $^a ** 2 }
    sub do-stuff-with(&lambda, $param) { lambda($param) }
    say do-stuff-with( &frobnicate, 42 );  # 1764

$ - Scalar vs Item
==================
Now that we've seen the `@`, `%` and `&` sigils, the `$` sigil is a bit
bland.  It does **not** enforce any type checks, so you can bind it to any
type of object.  So what *does* happen when you write:

    # Perl 6
    my $answer = 42;

Well, functionally something like this happens:

    # Perl 6
    my $answer := Scalar.new(42);

except that this happens at a very low level, so the above code will actually
**not** work, in case you wondered.  And that's all there is to it when your
doing declarations of scalar variables.

In Perl 6 the `$` also indicates that whatever is in there, should be
considered a single **item**.  So, even if a Scalar container has an
`Array` object, it will be considered as a single item in situations
where iteration is required:

    # Perl 6
    my @foo = 1,2,3;
    my $bar = Array.new(1,2,3);  # alternately: [1,2,3]
    .say for @foo;  # 1␤2␤3␤
    .say for $bar;  # [1 2 3]

Note that in the latter case, only **one** iteration is done.  And in the
former case **three**.  Whether you want something to iterate or not, can
be used by prefixing the appropriate sigil:

    # Perl 6
    .say for $@foo;  # [1 2 3] , consider the array as an item
    .say for @$bar;  # 1␤2␤3␤  , consider the scalar as a list

Although maybe that brings us too much into line-noise land.  Fortunately,
there are more verbose equivalents:

    # Perl 6
    .say for @foo.item;  # [1 2 3] , consider the array as an item
    .say for $bar.list;  # 1␤2␤3␤  , consider the scalar as a list

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

Please do **not** confuse the * used to indicate slurpiness of parameters in
Perl 6 with the typeglob sigil of Perl 5: they have nothing to do with each
other.

Sigilless variables
===================
Perl 5 does not support sigilless variables out of the box (apart from maybe
using left-value subroutines, but that would be very clunky indeed).

Perl 6 does not directly support sigilless **variables** either, but it does
support **binding** to sigilless names by prefixing a backslash ("\\") to the
name in a definition:

    # Perl 6
    my \the-answer = 42;
    say the-answer;  # 42

Since the right-hand side of the assignment is a constant, this is basically
the same as defining a constant:

    # Perl 5
    use constant the_answer => 42;
    say the_answer;  # 42

    # Perl 6
    my constant the-answer = 42;
    say the-answer;  # 42

It becomes more interesting if the right hand side of a definition is
something else.  Something like a container!  This allows for the following
syntactic trick to get sigilless variables:

    # Perl 6
    my \foo = $ = 41;                # a sigilless scalar variable
    my \bar = @ = 1,2,3,4,5;         # a sigilless array
    my \baz = % = a => 42, b => 666; # a sigilless hash

This basically creates nameless entities (a scalar, an array and a hash),
initializes them using the normal semantics, and then **binds** the resulting
objects (a `Scalar` container, an `Array` object and a `Hash` object) to the
sigilless name.  Which you can then use as any other ordinary variable in
Perl 6.

    # Perl 6
    say ++foo;     # 42
    say bar[2];    # 3
    bar[2] = 42;
    say bar[2];    # 42
    say baz<a b>;  # (42 666)

Of course, if you do this, you will lose all of the advantages of sigils,
specifically with regards to interpolation.  You will then basically always
need to use **{ }** in interpolation.

    # Perl 6
    say "The answer is {the-answer}.";  # The answer is 42.

Which would be rather more cumbersome in most versions of Perl 5:

    # Perl 5
    say "The answer is @{[the_answer]}.";  # The answer is 42.

Summary
-------
All variables in Perl 6 could be considered **tied** variables when thinking
about them using Perl 5 concepts.  This makes them slowish initially.  But
runtime optimization and JITting of hot code paths (at one point to machine
code), already makes them faster than Perl 5 variables in some benchmarks.

The `@`, `%` and `&` in Perl 6 do not create any specific objects, but rather
indicate a type constraint that will be applied to the object a name will be
bound to.  The `$` sigil is different in that respect, as there is no type
constraint to be enforced.

The `@` and `$` prefixes indicate listification and itemization respectively,
although it's probably more readable to use the `.list` and `.item` methods
instead.

With a little syntactic trick, you **can** program your Perl 6 programs
without using any sigils in variable names.
