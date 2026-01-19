# Cases of UPPER

The [Raku Programming Language](https://raku.org) provides quite a few ways of modifying the behaviour of code execution, and interfaces to classes and objects, by means of syntax elements in UPPERCASE.

This series of blog posts intends to provide an overview of these syntax elements, and how they can be used to your benefit.

Why not read the documentation?  Well, in some cases there is ample documentation.  For instance in the case of [phasers](https://docs.raku.org/language/phasers).  But in other cases the documentation is less clear.  Especially if it is about when to use a given feature to provide a certain capability to your code.

Nonetheless it may be a good idea to start with (maybe yet another) overview of phasers, their flavours and how you can use them.

## By execution stage

In the Raku Programming Language there are basically two stages of execution (that matter for you as a developer): "compile time" and "runtime".

The compile time stage takes care of parsing the Raku code into executable bytecode.  However, it is **also** possible to actually **execute** code during compilation.  One of the ways to do that, is with the `BEGIN` phaser.

As with most phasers, the `BEGIN` phaser can either take a `Block` or a so-called `thunk` (a piece of code that shares its scope with the surrounding scope, and that will **not** be executed immediately).
```raku
say "after";
BEGIN say "before";
```
will display:
```
before
after
```
because the `say "before"` is executed at compile time, and the `say "after"` at runtime.

Note that these `say`s occurred in "thunk"s because of the lack of `{ }` indicating a scope.  If you have multiple lines of code you need to execute at compile time, you can do that in such a scope:
```raku
BEGIN {
    my $now = DateTime.now;
    say $now;
}
```
will show when the code is being compiled.

The `BEGIN` phaser also returns the last value seen in it.  And you can store that in a variable that will be set at compile time:
```raku
my $compiled-at = BEGIN DateTime.now;
say "This code was compiled at: $compiled-at";
```

