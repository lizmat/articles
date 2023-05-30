Shaking the RakuAST Tree
========================
RakuAST for Early Adopters (part 5)

As we've seen in the previous installment, one of the interesting things about ASTs in general is that you can walk the tree to look for certain objects, or combination of objects, and act accordingly.  And that action can actually be a modification!

But before we go on:

Long names shorter
------------------
One reaction to the [second installment](https://dev.to/lizmat/a-practical-example-of-rakuast-18jk) of this series, was the length of the class names used (specifically `RakuAST::Regex::CharClassEnumerationElement::Character`): couldn't the class names be made shorter?

And the answer is *no* and *yes*.  No, because when you're creating upto 400 classes in a new hierarchy, the semantics of a class should be very clear from its name.  It was a conscious decision to **not** make shorter names.

However, the answer is also *yes* because in the Raku Programming Language you can specify an alias to **any** class name using [`constant`](https://docs.raku.org/language/variables#The_constant_prefix).  Suppose you want to change the prefix `RakuAST::` to `R`, and `RakuAST::Regex::` to `Rx::`, etc. etc.:
```
constant R      = RakuAST;
constant Rx     = R::Regex;
constant Rxccee = Rx::CharClassEnumerationElement;
```
or do it in one go:
```
constant Rxccee = RakuAST::Regex::CharClassEnumerationElement;
```
After either of these you will be able to refer to `RakuAST::Regex::CharClassEnumerationElement::Character` as `Rxccee::Character`.  Does that make your code more readable?  Perhaps.  These `constant` definitions are by default [`our`](https://docs.raku.org/language/variables#The_our_declarator).  It is usually better to prefix them with [`my`](https://docs.raku.org/language/variables#The_my_declarator), so that you can **only** use them inside the scope where they are defined.  Taking the "char-matcher" example again, but this time defining a `Rx` constant **inside** the subroutine:
```
sub chars-matcher($string) {
    my constant Rx = RakuAST::Regex;
    my @elements = $string.comb.unique.map: {
        Rx::CharClassEnumerationElement::Character.new($_)
    }
    RakuAST::TokenDeclaration.new(
      body => Rx::Assertion::CharClass.new(
        Rx::CharClassElement::Enumeration.new(:@elements)
      )
    ).EVAL
}
```
By defining it inside the subroutine only, you make sure that such shortcuts do not leak out, and that you have the "explanation" of the short-cut easily available.

Optimizing the tree
-------------------
One of the most simple types of optimization a programming language can do, is [constant folding](https://en.wikipedia.org/wiki/Constant_folding).  So let's see a simple example of this at work:
```
my $a = 42 + 666 + 137;
```
This is clearly an expression that consists of only literal (constant) values, so it should be possible to simplify this to `my $a = 845`.  This can be done by adding a `CHECK` phaser that will look at the AST, and do the necessary changes:
```
my $a = 42 + 666 + 137;

CHECK {
    say $*CU.statement-list.statements[0].DEPARSE;  # my $a = 42 + 666 + 137
    for $*CU -> $ast {
        if $ast ~~ RakuAST::ApplyInfix {
            my $parent := @*LINEAGE[0];
            if $parent.can("set-expression") {
                with $ast.literalize {
                    $parent.set-expression(RakuAST::Literal.new($_))
                }
            }
        }
    }
    say $*CU.statement-list.statements[0].DEPARSE;  # my $a = 845
}
```
The first line inside the `CHECK` phaser shows the code of the line we want to optimize.  To make it easier to read, I've decided to show the Raku source representation (`.DEPARSE`).  The ` $*CU.statement-list.statements[0]` selects the first statement in the statement list of the compilation unit (which is the first line in this example).  Which would show: "my $a = 42 + 666 + 137"

Then all objects in the AST will be walked with `for $*CU -> $ast {`.  This works, because RakuAST nodes have a specialized `.map` method, and `for` is nothing other than `.map` in a sink context (aka: discarding any result from the `.map`).

When a `RakuAST::ApplyInfix` object is encountered (`if $ast ~~ RakuAST::ApplyInfix {`), then the parent of this RakuAST object in the AST will be saved (from the `@*LINEAGE` dynamic array, which contains all parents of the object).

Then if it is possible to alter the expression in that parent object (by checking if the object can execute a `set-expression` method, aka`if $parent.can("set-expression") {`), an attempt is made to create a literal (constant) value for the current object (`with $ast.literalize {`).

If that is successful, then a new `RakuAST` object is created for that value (`RakuAST::Literal.new($_)`) and is then used to replace the expression in the parent (`$parent.set-expression()`).

Then we show the source representation of the first line again: which is now "my $a = 845", showing that our constant folding was successful!

Now that you've seen this, the question becomes: would you as a user of Raku, need to do this?  And the answer is **no**.  These types of optimizations will be built into Rakudo, so you don't need to worry about it.  This was just an example of how this could work internally, and perhaps give you some visions of evil sugarplums dancing in your head to do other types of introspection and/or modifications!

Conclusion
----------
This installment shows how you can shorten long `RakuAST::` class names within a scope by using `my constant`.  It also shows an example of actual constant folding.

The intended audience are those people willing to be early adopters of these exciting new features in the Raku Programming Language.  The examples in this blog post will work in the next release of the Rakudo compiler (probably 2023.06), but are now already available in the bleeding edge version.

This is also the last installment of this series: other aspects of RakuAST will be handles in different, more focused on specific RakuAST characteristics.
