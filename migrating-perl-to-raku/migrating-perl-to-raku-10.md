# Sigils (Part 1 of 2)
In this blog post we will look at the subtle differences in [sigils](https://en.wikipedia.org/wiki/Sigil_(computer_programming)) (the symbols at the start of a variable name) between Perl and Raku.

## An overview
Let's start with an overview of sigils in Perl and Raku:
```
  Sigil   Perl         Raku
  --------------------------------
    @     Array        Positional
    %     Hash         Associative
    &     Subroutine   Callable
    $     Scalar       Item
    *     Typeglob     n/a
  --------------------------------
```

# @ (Array vs. Positional)
When you define an array in Perl, you create an expandable list of scalar values and give it a name with the sigil `@`:
```
# Perl
my @foo = (1,2,3);
push @foo, 42;
say for @foo;  # 1␤2␤3␤42␤
```
When you define an array in Raku, you create a new [`Array`](https://docs.raku.org/type/Array) object and bind it to the entry by that name in the lexical pad. So:
```
# Raku
my @foo = 1,2,3;
push @foo, 42;
.say for @foo;  # 1␤2␤3␤42␤
```
is functionally the same as in Perl. However, the first line is syntactic sugar for:
```
# Raku
my @foo := Array.new( 1,2,3 );
```
This binds (rather than assigns) a new `Array` object to the lexically defined name `@foo`. The `@` sigil in Raku indicates a type constraint: if you want to bind something into a lexpad entry with that sigil, it must perform the [`Positional`](https://docs.raku.org/type/Positional) role.

It's not hard to determine whether a class performs a certain role using smartmatch:
```
# Raku
say Array ~~ Positional;   # True
```
You could argue that all arrays in Raku are implemented in the same way as `tied arrays` are implemented in Perl. And that would not be far from the truth.

Without going too deep into the specifics, a simple example might clarify this. The `AT-POS` method is one of the key methods of a class implementing the `Positional` role. This method is called whenever a single element needs to be accessed.

So, when you write:
```
# Raku
say @a[42];
```
you are actually executing:
```
# Raku
say @a.AT-POS(42);
```
Actually, it's slightly more complex than this, but in essence this is correct.

Of course, this is not the only method you could implement; there are many more.  The [`Array::Agnostic`](https://raku.land/zef:lizmat/Array::Agnostic) module makes this very easy.

Rather than having to bind your class performing the `Positional` role, there's a special syntax using the `is` trait. So instead of having to write:
```
# Raku
my @a := YourClass.new( 1,2,3 );
```
you can write:
```
# Raku
my @a is YourClass = 1,2,3;
```
In Perl, tied arrays are notoriously slow compared to "normal" arrays. In Raku, arrays are similarly slow at startup.

Fortunately, Rakudo Raku optimises hot-code paths by inlining and "just in timing" (JITting) opcodes to machine code where possible. (Thanks to advancements in the optimiser, this happens sooner, more often, and better).

# % (Hash vs. Associative)
Hashes in Raku are implemented similar to arrays; you could also consider them a tied hash (using Perl terminology).

Instead of the `Positional` role used to implement arrays, the `Associative` role should be used to implement hashes.  Again, a simple example might help to understand this better.

The `AT-KEY` method is one of the key methods of a class implementing the `Associative` role. This method is called whenever the value of a specific key needs to be accessed.

So, when you write:
```
# Raku
say %h<foo>;
```
you are executing:
```
# Raku
say %h.AT-KEY("foo");
```
There are many other methods you can implement to create custom behaviour of hashes and arrays in Raku.  The [`Hash::Agnostic`](https://raku.land/zef:lizmat/Hash::Agnostic) module helps you with doing that.

# & (Subroutine vs. Callable)
In Perl, there is only one type of callable executable code, the subroutine:
```
# Perl
sub frobnicate { shift ** 2 }
```
And, if you want to pass on a subroutine as a parameter, you need to get a reference to it:
```
# Perl
sub do_stuff_with {
    my $lambda = shift;
    &$lambda(shift);
}
say do_stuff_with( \&frobnicate, 42 );  # 1764
```
In Raku, multiple types of objects can contain executable code. What they have in common is that they consume the `Callable` role.

The `&` sigil forces binding to an object performing the [`Callable`](https://docs.raku.org/type/Callable) role, just like the `%` sigil does with the `Associative` role and the `@` sigil does with the `Positional` role. An example very close to Perl is:
```
# Raku
my &foo = sub ($a,$b) { $a + $b }
say foo(42,666);  # 708
```
Note that even though the variable has the `&` sigil, you do not need to use it to execute the code in that variable.

In fact, if you ran the code in a `BEGIN` block, there would be no difference compared to an ordinary sub declaration:
```
# Raku
BEGIN my &foo = sub ($a,$b) { $a + $b } # same as sub foo()
```
In contrast to Perl, in Raku a `BEGIN` block can be a single statement without a block.  This allows it to share its lexical scope with the outside scope. But more on that later.

The main advantage to using a variable with the `&` sigil is that it will be known at compile time that there will be something executable in there, even if that something isn't yet known.

There are other ways to set up a piece of code for execution:
```
# Raku
my &boo = -> $a, $b { $a + $b } # a Block with a signature
my &goo = { $^a + $^b }         # auto-generated signature
my &woo = * + *;                # Whatever currying
```
If you'd like to know more, please see:
- [Block](https://docs.raku.org/type/Block) with a signature
- [Autogenerated signatures](https://docs.raku.org/language/variables#The_^_twigil)
- [Whatever currying](https://docs.raku.org/type/Whatever)

The one you use depends on the situation and your preferences.

Finally, you can also use the `&` sigil inside a signature to indicate that the callee wants something executable there. Which brings us back to the first two code examples in this blog post:
```
# Perl
sub frobnicate { shift ** 2 }
sub do_stuff_with {
    my $lambda = shift;
    &$lambda(shift)
}
say do_stuff_with( \&frobnicate, 42 );  # 1764
```
```
# Raku
sub frobnicate { $^a ** 2 }
sub do-stuff-with (&lambda, $param) {
    lambda($param)
}
say do-stuff-with( &frobnicate, 42 );   # 1764
```
Note that in Raku you don't need to take a reference; you can simply pass the code object (as indicated by the `&` sigil) as an argument.

## Summary
All variables in Raku could be considered tied variables when thinking about them using Perl concepts. This makes them somewhat slow initially. But runtime optimisations and JITting of hot-code paths (at one point to machine code) already make it faster than Perl variables in some benchmarks.

The `@`, `%`, and `&` sigils in Raku do not create any specific objects, rather they indicate a type constraint that will be applied to the object a name is bound to.

More about the `$` sigil in Sigils (Part 2).
