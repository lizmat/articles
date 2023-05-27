So why is there RakuAST in the first place?
===========================================
RakuAST for Early Adopters (part 3)

Several people have asked me to explain why RakuAST is such a big thing.  And why it was deemed necessary to go down this way.  I will try to explain this in this blog post.  Without getting too technical I hope.

There are several stages your Raku program goes through before it becomes executable bytecode that can actually be run by a virtual machine such as [MoarVM](https://moarvm.org) or the JVM.

Before RakuAST
--------------
A highly simplified explanation: the first step is to parse your program using a [grammar](https://docs.raku.org/language/grammar_tutorial).  The grammar determines whether the *syntax* of your program is correct.  For instance: 'say "Hello World";' is syntactically correct and interpreted as a statement (because of the trailing `;`) with the argument `"Hello World"` being passed to something callable called `say`.

While the parsing of your program is progressing, a data structure is built through the the calling of the associated [action methods](https://docs.raku.org/language/grammar_tutorial#Grammar_actions).  That data structure stores the name of the callable `say`, and its argument `"Hello World"` in such a way that it can be used for the next step.

Once your Raku program has been parsed completely, the data structures that were built up during parsing, are used to create the bytecode that can actually be run.  And then usually execute that bytecode, unless we were pre-compiling a module, in which case the bytecode is saved in a file.

> If you really want to look at this, you can find the code in [src/Perl6/Grammar.nqp](https://github.com/rakudo/rakudo/blob/main/src/Perl6/Grammar.nqp), [src/Perl6/Actions.nqp](https://github.com/rakudo/rakudo/blob/main/src/Perl6/Actions.nqp) and [src/Perl6/World.nqp](https://github.com/rakudo/rakudo/blob/main/src/Perl6/World.nqp).

After RakuAST
-------------
Until RakuAST, these data structures were just that: data structures that only could be manipulated if you really knew how.  And that required knowledge of the way these data-structures were set up, and the methods that you could use to manipulate them.  Since these were considered internal information, the format of these data structures could change from one release of Rakudo to the next, rendering any third party modules that used these feaures inoperable.

With RakuAST, these data structures have been replaced by a set of [`RakuAST::`](https://docs.raku.org/type/RakuAST) objects, with a (not yet) documented interface.  But an interface that is intended to be stable, like all other Raku Programming Language features.  So it has [tests](https://github.com/rakudo/rakudo/tree/main/t/12-rakuast) that will ultimately be part of [roast](https://github.com/raku/roast#readme).  To facilitate development with RakuAST, and of RakuAST itself, there already are a number of  utility methods such as `.raku`, `.DEPARSE`, `.literalize` and of course, the `.AST` method on strings to have RakuAST objects created for you.

Also with RakuAST, there is now a complete separation of steps: whereas in the old situation sometimes the precursor to bytecode would already be created, this does not happen with RakuAST.  At the end of the parsing stage,
there is a single `RakuAST::CompUnit` object that contains a representation of **all** of the parsed code, including any documentation and declarator doc blocks.

This representation can then be altered if necessary (think [`macro`](https://docs.raku.org/language/experimental#macros), but then better!).  Or the representation can be trimmed to only contain documentation, if you just want to create HTML of the documentation.
```
# create the RakuAST tree of a given source file
my $ast = $filename.IO.slurp.AST;
# show the ASTs of Rakudoc objects in that source file
.say for $ast.rakudoc;
```
Or it can be used to already do some optimizations, such as [constant folding](https://en.wikipedia.org/wiki/Constant_folding).  A *very*  simple example of constant folding:
```
# fold simple integer addition into a single integer
sub fold-integers($ast) {

    # is it a + operation?
    if $ast ~~ RakuAST::ApplyInfix && $ast.infix.operator eq '+' {
        my $left  = $ast.left;
        my $right = $ast.right;

        # integers on both sides?
        if $left ~~ RakuAST::IntLiteral && $right ~~ RakuAST::IntLiteral {

            # We can calculate the result now, return as a single AST
            return RakuAST::IntLiteral.new(
              $left.compile-time-value + $right.compile-time-value
            )
        }
    }

    # no optimization possible
    $ast
}

# create an example AST
my $ast = "42 + 666".AST.statements.head.expression;
say $ast;
# RakuAST::ApplyInfix.new(
#   left  => RakuAST::IntLiteral.new(42),
#   infix => RakuAST::Infix.new("+"),
#   right => RakuAST::IntLiteral.new(666)
# )

say fold-integers($ast);
# RakuAST::IntLiteral.new(708)
```
Or it can be used to provide run-time optimizers with more information, such as improved [escape analysis](https://en.wikipedia.org/wiki/Escape_analysis).  All of these that would **not** be possible (or at least magnitudes of difficulty harder) without RakuAST.

BEGIN
-----
Of course, there is one feature that complicates matters: explicit and implicit [`BEGIN`](https://docs.raku.org/language/phasers#BEGIN) blocks.  Any code in these will be parsed, turned into RakuAST objects, converted to bytecode and executed **during** parsing of your program.
```
# explicit BEGIN
BEGIN say "compilation started at " ~ DateTime.now;

# implicit BEGIN
my constant compiled-at = DateTime.now;
```
It's like a program inside a program, of which the result can be saved for later reference.  And yes, you can do an [`EVAL`](https://docs.raku.org/type/independent-routines#routine_EVAL) inside a `BEGIN` block.  So it truly is a program inside a program: all of the features of the Raku Programming Language are available.

But now that we have RakuAST objects, it will actually be technically possible to inspect and possibly alter the RakuAST object tree at that time as well.  Which will also open some interesting future capabilities!  At the expense of severe headaches for the core developers :-)

Conclusion
----------
This installment gives a little background on why the development of RakuAST is so important for the future development of the Raku Programming Language.

The intended audience are those people willing to be early adopters of these exciting new features in the Raku Programming Language.
