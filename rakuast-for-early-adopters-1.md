RakuAST for Early Adopters (part 1)
===================================

Originally, I thought I'd name this series of blog posts "RakuAST for Beginners".  But since documentation on RakuAST is pretty non-existent at this stage, I felt I would be doing people a service by making clear that they will be at the very *front* of development in the [Raku Programming Language](https://raku.org) if they start dabbling in RakuAST.

What Is It?
-----------
If you're a programmer, you may be aware of the concept of an [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree).  That's where the "AST" in RakuAST comes from.

Normally, when you write a program in the Raku Programming Language, all of the business of compiling your code into something that can be executed, is done for you under the hood.  One of the steps in this process is to create an Abstract Syntax Tree (aka **AST** from now on) from your code, and convert that to bytecode that an interpreter (or "virtual machine") such as [MoarVM](https://moarvm.org) can execute.

So where does RakuAST come into this?  Well, RakuAST allows you to create an AST (that can be converted to bytecode and run) programmatically **without** needing to create intermediate source code.

How ready is it for primetime?
------------------------------
Not (yet).  Several areas of RakuAST features and semantics are still un(der)developed.  But there is enough implemented to allow the new "Raku" grammar to handle Raku source code well enough to make 64% of Raku test files pass completely.  A lot of the work that needs to be done with the "Raku" grammar, is to use the already existing RakuAST features, rather than needing new features in RakuAST itself.  But that is worth another series of blog posts in itself.

> If you want to run your code with the new Raku grammar, you must set the `RAKUDO_RAKUAST=1` environment variable *before* running.  Otherwise the default grammar (which is now referred to as the "legacy" grammar) will be used.

In any case, because it is not ready for primetime (yet), and some interfaces and semantics might still change, one will have to put a `use experimental :rakuast` in the code.  Or indicate you want the current development language version, by putting a `use v6.e.PREVIEW` in the code.

So when should I use this?
--------------------------
When it is handy for you to do so.

To give you an example: [`sprintf`](https://docs.raku.org/type/independent-routines#routine_sprintf) takes a format string to create a string representation of the values given.  This is currently implemented with a special "format-string" grammar.  Everytime a `sprintf` is executed (either directly, or by using [`printf`](https://docs.raku.org/type/independent-routines#routine_printf) or [`.fmt`](https://docs.raku.org/type/List#method_fmt)), it will parse the format using that grammar.  And the associated actions then produce the string representation for the given values.

Needless to say, this is very repetitive and cpu-intensive.  Wouldn't it be better to only parse the format *once*, and then create code for that, and run that *code* everytime?

Yes, it would.  But until there was RakuAST, that was virtually impossible to do because there was no proper API for building ASTs.  Nor was there an interface to execute those ASTs.  And now that there is RakuAST, it is actually possible to do this.  And there **is** actually already an implementation of that idea in the new [Formatter](https://github.com/rakudo/rakudo/blob/main/src/core.e/Formatter.pm6) class.  Although this is definitely **not** intended as an entry point into grokking RakuAST.

But maybe a better way to tell whether it is handy for you to use RakuAST, is to use RakuAST whenever you need to resort to using [`EVAL`](https://docs.raku.org/type/independent-routines#routine_EVAL).  Because with RakuAST, you will have a way to not have to worry about incidental code insertion, and you will be able to create semantics for which there is no way in the Raku Programming Language (yet).

Let's start at the beginning
----------------------------
It is common to start with a "Hello World", so let's start with one here as well.  This is the syntax to create an AST for saying "Hello World":
```
use experimental :rakuast;

my $ast = RakuAST::Call::Name.new(
  name => RakuAST::Name.from-identifier("say"),
  args => RakuAST::ArgList.new(
    RakuAST::StrLiteral.new("Hello world")
  )
);
```
This creates a RakuAST tree and puts that in the `$ast` variable.  There is no output yet, because all that was done here, was to *create* the RakuAST objects.  To actually convert to bytecode and run that, one needs to call the `.EVAL` method on the RakuAST object:
```
$ast.EVAL;  # Hello World
```
Pretty neat, eh?  But that's not all.  To help in development and debugging, you can `say` the `.raku` method on a RakuAST object, and it will create a representation of the object as *RakuAST objects* in Raku code.
```
say $ast.raku;
# RakuAST::Call::Name.new(
#   name => RakuAST::Name.from-identifier("say"),
#   args => RakuAST::ArgList.new(
#     RakuAST::StrLiteral.new("Hello world")
#   )
# );
```
And since most uses of `say`ing RakuAST objects will be to see this representation, you can actually drop the `.raku` part there, so `say $ast` will give you the same output.

Of course, sometimes you would like to see how a RakuAST object would look like as Raku *source code*.  And there's a method for that as well: `.DEPARSE`:
```
say $ast.DEPARSE;  # say("Hello World")
```
Of course, the `.DEPARSE` output will be a little more formal than original.  But it will (usually) be legal source code, and round-trippable.  And you could argue that this could be used as a (simple) linter.  And you'd be right: the way `.DEPARSE` is implemented, is that it is pluggable (so one could implement their own way of deparsing RakuAST objects).  But that in itself is again enough for a series of blog posts.

Finally, sometimes you would have a piece of source code of which you would like to know the RakuAST representation.  And for that, there's the `.AST` method on strings:
```
my $ast = 'say "Hello World"'.AST;
say $ast;
# RakuAST::StatementList.new(
#   RakuAST::Statement::Expression.new(
#     expression => RakuAST::Call::Name.new(
#       name => RakuAST::Name.from-identifier("say"),
#       args => RakuAST::ArgList.new(
#         RakuAST::QuotedString.new(
#           segments   => (
#             RakuAST::StrLiteral.new("Hello World"),
#           )
#         )
#       )
#     )
#   )
# )
```
Note that this is slightly more complex than the initial example.  But you hopefully see that that's because this is now wrapped as an expression in a statement, which is part of a statement list.  And the double quoted string hasn't been flattened yet.

And it should also be noted that this functionality depends on the "Raku" grammar, which does not yet support all Raku Programming Language functionality.  So in some cases, it may still not do what you hoped it would do.

Conclusion
----------
This blog post introduces RakuAST, an interface to create Abstract Syntax Trees in the Raku Programming Language.  It shows how to build a "Hello World" AST, and shows how to run the AST, how it was created and how it could be represented as Raku source code.

The intended audience are those people willing to be early adopters of these exciting new features in the Raku Programming Language.
