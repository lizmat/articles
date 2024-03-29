# Containers Vs References
In this blog post we'll focus on Perl's references and how they're handled in Raku, and introduce the concepts of binding and containers.

## References
There are *no* references in the Raku Programming Language, which is surprising to many people used to Perl's semantics. But worry not: because there are no references, you *don't* have to worry about whether something should be de-referenced or not.
```
# Perl
my $foo = \@bar;   # must add reference \ to
                   # make $foo a reference to @bar
say $bar[1];       # no dereference needed
say $foo->[1];     # must add dereference ->
```
```
# Raku
my $foo = @bar;    # $foo now contains @bar
say @bar[1];       # no dereference needed,
                   # note: sigil does not change
say $foo[1];       # no dereference needed either
```
One could argue that everything in Raku is a reference. Coming from Perl (where an object is a blessed reference), this would be a logical conclusion about Raku where everything is an object (or can be considered one). But that wouldn't do justice to the situation in Raku and would hinder you in understanding how things work in Raku. Beware of [false friends](https://en.wikipedia.org/wiki/False_friend)!

## Binding
Before we get to assignment, it is important to understand the concept of binding in Raku. You can bind something explicitly to something else using the [`:=`](https://docs.raku.org/language/operators#infix_:=) operator. When you define a lexical variable, you can bind a value to it:
```
# Raku
my $foo := 42;  # note: := instead of =
```
Simply put, this creates a key with the name `$foo` in the lexical pad (lexpad) (which you could consider a compile-time hash that contains information about things that are visible in that lexical scope) and makes 42 its literal value.  Because this is a literal constant, you can't now write `$foo = 99;` to change it, just like you can't write `42 = 99;`.  Trying to do so will cause an exception.

This binding operation is used under the hood in many situations, for instance when iterating:
```
# Raku
my @a = 0..9;  # can also be written as ^10
say @a;        # [0 1 2 3 4 5 6 7 8 9]
for @a {
    $_++;      # $_ is bound to each array element
}              # and incremented
say @a;        # [1 2 3 4 5 6 7 8 9 10]
```
If you try to iterate over a constant list, then `$_` is bound to the literal values, which you can *not* increment:
```
# Raku
for 0..9 {
    $_++;  # error: requires mutable arguments
}
```
## Assignment
If you compare "create a lexical variable and assign to it" in Perl and Raku, it looks the same on the outside:
```
my $bar = 56;  # both Perl and Raku
```
In Raku, this also creates a key with the name `$bar` in the lexpad. But instead of directly binding the value to that lexpad entry, a [container](https://docs.raku.org/language/containers) (a [`Scalar`](https://docs.raku.org/type/Scalar) object) is created for you and that is bound to the lexpad entry of `$bar`. Then 56 is stored as the value in that container.

In pseudo-code, you can think of this as:
```
# Pseudocode
my $bar := Scalar.new( value => 56, name => '$bar' );
```
Notice that the `Scalar` object is bound, not assigned. The closest thing to this in Perl is a tied scalar. But of course `= 56` is much less to type!

Data structures such as `Array` and `Hash` also automatically put values in containers bound to the structure.
```
# Raku
my @a;       # empty Array
@a[5] = 42;  # bind a Scalar container to 6th element
             # and put 42 in it
```

## Containers
The `Scalar` container object is invisible for most operations in Raku, so most of the time you don't have to think about it.

For instance, whenever you call a subroutine (or a method) with a variable as an argument, it will bind to the value in the container. And because you cannot assign to a value, you get:
```
# Raku
sub frobnicate($this) {
    $this = 42;
}
my $foo = 666;
frobnicate($foo); # Cannot assign to a readonly variable or a value
```
If you want to allow assigning to the outer value, you can add the [`is rw`](https://docs.raku.org/language/signatures#Parameter_traits_and_modifiers) trait to the variable in the signature. This will bind the variable in the signature to the container of the variable specified, thus allowing assignment:
```
# Raku
sub oknicate($this is rw) {
    $this = 42;
}
my $foo = 666;
oknicate($foo); # no problem
say $foo;       # 42
```

## Proxy
Conceptually, the `Scalar` object in Raku has a `FETCH` method (for producing the value in the object) and a `STORE` method (for changing the value in the object), just like a tied scalar in Perl.

Suppose you later assign the value 768 to the `$bar` variable:
```
# Raku
$bar = 768;
```
What happens is conceptually the equivalent of:
```
# Raku
$bar.STORE(768);
```
Suppose you want to add 20 to the value in `$bar`:
```
# Raku
$bar = $bar + 20;
```
What happens conceptually is:
```
# Raku
$bar.STORE( $bar.FETCH + 20 );
```
If you like to specify your own `FETCH` and `STORE` methods on a container, you can do that by binding to a [`Proxy`](https://docs.raku.org/type/Proxy) object. For example, to create a variable that will always report twice the value that was assigned to it:
```
# Raku
my $double := do {  # $double now a Proxy,
                    # rather than a Scalar container
    my $container;  # the actual container
    Proxy.new(
      FETCH => method ()       { $container + $container },
      STORE => method ($value) { $container = $value }
    )
}
$double = 42;
say $double;  # 84
```
Note that you will need an extra variable (in this example: `$container`) to keep the value.

## Constraints and default
Apart from the value, a `Scalar` object also contains extra information such as the type constraint and default value. Take this definition:
```
# Raku
my Int $baz is default(42) = 666;
```
It creates a `Scalar` bound with the name `$baz` to the lexpad, constrains the values in that container to types that successfully smartmatch with `Int`, sets the default value of the container to 42, and puts the value 666 in the container.

Assigning a string to that variable will fail because of the type constraint:
```
# Raku
$baz = "foo";
# Type check failed in assignment to $baz;
#   expected Int but got Str (“foo")
```
If you do not give a type constraint when you define a variable, then the [`Any`](https://docs.raku.org/type/Any) type will be assumed. If you do not specify a default value, then the type constraint will be assumed.

Assigning [`Nil`](https://docs.raku.org/type/Nil) (the Raku equivalent of Perl's `undef`) to that variable will reset it to the default value:
```
# Raku
say $baz;   # 666
$baz = Nil;
say $baz;   # 42
```

## Summary
Perl has values and references to values. Raku has no references, but it has values and containers. There are two types of containers in Raku: `Proxy` (which is much like a tied scalar in Perl) and `Scalar`. Simply stated, a variable, as well as an element of a `List`, `Array`, or `Hash`, is either a value (if it is bound), or a container (if it is assigned).

Whenever a subroutine (or method) is called, the given arguments are de-containerised and bound to the parameters of the subroutine (unless told to do otherwise). A container also keeps information such as type constraints and a default value. Assigning `Nil` to a variable will return it to its default value, which is `Any` if you do not specify a type constraint.
