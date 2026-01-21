# Cases of UPPER

The [Raku Programming Language](https://raku.org) contains more than **50** syntax elements that are completely in UPPERCASE.  Why are they in uppercase?  Because they indicate "Something Very Strange" going on (hence the allcaps).

These syntax elements provide quite a few ways of modifying the behaviour of code execution,  and interfaces to classes and objects.

This series of blog posts intends to provide an overview of these syntax elements (which can loosely be grouped into "Phasers", "Methods", "Macros" and "Pragmas"), and how they can be used to your benefit.

Why not read the documentation?  Well, in some cases there is ample documentation.  For instance in the case of [phasers](https://docs.raku.org/language/phasers).  But in other cases the documentation is less clear.  Especially if it is about when to use a given feature to provide a certain capability to your code.

Nonetheless it may be a good idea to start with (maybe yet another) overview of phasers, their flavours and how you can use them.  Just to get a bit of a feel for it!

## Phasers, by execution stage

In the Raku Programming Language there are basically two stages of execution (that matter for you as a developer): "compile time" and "runtime".

The compile time stage takes care of parsing the Raku code into an AST (which is short for [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree).  When the compilation is complete, then that AST is converted into bytecode.

And the runtime stage is when that bytecode is actually being executed by the virtual machine (in case of Rakudo, that is most commonly [`MoarVM`](https://moarvm.org/features.html).

### Compile time

When a Raku program is being compiled, it is **also** possible to actually **execute** code during this compilation.  One of the ways to do that, is with the [`BEGIN` phaser](https://docs.raku.org/syntax/BEGIN).

#### BEGIN

As with most phasers, the `BEGIN` phaser can either take a `Block` (`{ ... }`) or a so-called `thunk`.

> A "thunk" is a piece of code that has no scope of its own (and thus shares it scope with the surrounding scope).
```raku
say "after";
BEGIN say "before";
```
will display:
```
before
after
```
because the `say "before"` is executed at compile time, and the `say "after"` at runtime.  Even though you might think otherwise when you look at the order in which they occur in the code.

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
> Another way to create constant values at compile time, is to use the [`constant`](https://docs.raku.org/language/terms#Constants) directive.  For instance `my constant $answer = 42`.  The thunk on the right-hand side of the `=` is evaluated at compile time.

So what makes more sense to use?  The `BEGIN` phaser, or the `constant` directive?  For a part that depends on your coding style, and another part on the complexity of the code that you want to execute at compile time.  And sometimes you [use them side-by-side](https://github.com/lizmat/Text-Emoji/blob/main/lib/Text/Emoji.rakumod#L4-L15).

> Note that the [`use` statement](https://docs.raku.org/syntax/use) also may execute code at compile time if the module used has *not* been pre-compiled yet, or has [pre-compilation disabled](https://docs.raku.org/syntax/precompilation).

#### CHECK

The [`CHECK` phaser](https://docs.raku.org/syntax/CHECK) gets executed once the entire source code has been compiled into an AST.  It comes both in `Block` and `thunk` flavours.

In the next language level of Raku, this will allow you to actually modify the AST before it is being turned into bytecode.  But we're not there yet.

It is of little use currently.  One could for instance use it to prevent actual execution, but only if the code was being compiled (which may *not* be the case if a module was already pre-compiled).
```raku
CHECK exit if $phase-of-the-moon;
```
If you really want to *always* prevent execution of your program or module depending on the phase of the moon, you can use the `INIT` phaser:
```raku
INIT exit if $phase-of-the-moon;
```
More on the `INIT` phaser in the next episode!

## Conclusion

There are about 50 language elements in the Raku Programming Language that consist of only UPPERCASE characters.  The `BEGIN` and `CHECK` phaser apply to the compilation stage of a Raku program.

This concludes the first episode of cases of UPPER language elements in the Raku Programming Language.  Stay tuned for more!
