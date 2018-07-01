Migrating Perl 5 code to Perl 6
===============================

In this post I will be focusing on how Perl 5 references are being handled
in Perl 6.

Part 2 - Containers
===================
There are *no* references in Perl 6.  This revelation usually comes as quite
a shock to many people used to the semantics of references in Perl 5.  But
worry not: because there are no references, you do not have to worry anymore
whether something should be de-referenced or not!

One could argue that *everything* in Perl 6 is a reference.  Coming from
Perl 5 (where an object is a blessed reference) looking at Perl 6 where
*everything* is an object (or can be considered as one), this would be a
logical conclusion.  But that would not do justice to the actual situation
in Perl 6, and would hinder you in understanding how things work in Perl 6.
Beware of [false friends](https://en.wikipedia.org/wiki/False_friend)!

Binding
-------
Before we get to assignment, it is important to understand the concept of
binding in Perl 6.  You can bind something explicitely to something else
using the `:=` operator.  So what happens if you define a lexical variable
and you bind a value to it, e.g.:

    my $foo := 42;

Simply put, this creates a key with the name "`$foo`" in the lexpad (which
you could consider a compile-time hash) and makes `42` its *literal* value.
Because this is a literal constant, you cannot change it.  Trying to do so
will cause an exception.

Assignment
----------
Now compare this when we create a lexical variable and *assign* to it:

    my $bar = 56;

This *also* creates a key, this time with the name "`$bar`" in the lexpad.
But insteading of binding the value, a `Scalar` object is created internally
with `56` as the value to be stored for the key "`$bar`".  In code, you can
think of this as:

    my $bar := Scalar.new(56);

Summary
-------
Perl 6 differentiates between values and containers.  There are 2 types of
container: [Scalar](https://docs.perl6.org/type/Scalar) and
[Proxy](https://docs.perl6.org/type/Proxy) (which is much like a tied scalar
in Perl 5).  Simply stated, a variable, as well as an element of a
[List](https://docs.perl6.org/type/List), 
[Array](https://docs.perl6.org/type/Array) or
[Hash](https://docs.perl6.org/type/Hash), is either a value (if it is
*bound*), or a container (if they are *assigned*).  Whenever a subroutine
(or method) is called, the given arguments are de-containerized and then
*bound* to the parameters of the subroutine (unless told to do otherwise).