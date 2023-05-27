A practical example of RakuAST
==============================
RakuAST for Early Adopters (part 2)

A while ago someone asked on #raku if it is possible to create a [Raku character class](https://docs.raku.org/language/regexes#Enumerated_character_classes_and_ranges) with the valid characters being supplied by a string.  This is *not* possible by default in Raku at the moment.  But it **is** possible using RakuAST!

Let's first see how one can create characters classes with RakuAST by applying the `.AST` method to an example.

> By the way, all of these examples assume that a `use experimental :rakuast;` or `use v6.e.PREVIEW` is active.
```
say 'my token {<[abc]>}'.AST.statements.head.expression;
```
What this does is
- create the AST (`.AST`) for an anonymous [`token`](https://docs.raku.org/language/grammars#Rules) with a charclass for the letters "a", "b" and "c"
- then skips to the first statement (`.statements.head`)
- then skips to its `.expression`

because the `.AST` returns a statement list, and we're only interested in the expression of the first statement.

The result is:
```
RakuAST::TokenDeclaration.new(
  body => RakuAST::Regex::Assertion::CharClass.new(
    RakuAST::Regex::CharClassElement::Enumeration.new(
      elements => (
        RakuAST::Regex::CharClassEnumerationElement::Character.new("a"),
        RakuAST::Regex::CharClassEnumerationElement::Character.new("b"),
        RakuAST::Regex::CharClassEnumerationElement::Character.new("c"),
      )
    )
  )
);
```
So it looks like each character in the charclass is a separate `RakuAST::Regex::CharClassEnumerationElement::Character` object.  With this knowledge, it is pretty easy to make a custom `token` for the characters in a given string.  Let's create a subroutine "chars-matcher" that will create a token with a charclass of the characters for a given string:
```
sub chars-matcher($string) {
    my @elements = $string.comb.unique.map: {
        RakuAST::Regex::CharClassEnumerationElement::Character.new($_)
    }
    RakuAST::TokenDeclaration.new(
      body => RakuAST::Regex::Assertion::CharClass.new(
        RakuAST::Regex::CharClassElement::Enumeration.new(:@elements)
      )
    ).EVAL
}
```
First we create an array "@elements" and fill that with an enumeration object for each unique char in the given string (`$string.comb.unique`).  And then we create the TokenDeclaration object as from the example, but with the `@elements` array as the specification of the characters.  And then we convert that into an actual usable `token` by running `.EVAL` on it.

An example of its usage:
```
my $matcher = chars-matcher("Anna Mae Bullock");
say "Tina Turner" ~~ $matcher;       # ｢n｣
say "Tina Turner" ~~ / $matcher+ /;  # ｢na ｣
```
As you can see, you can use the generated `token` direct in a smart-match.  Or you can use it as part of a more complicated regex.

If you're ready to further dabble with RakuAST, it is probably a good idea to know a little bit of how it is currently being implemented.  So let's dive a little bit into that, to better understand some of the errors you might encounter when writing Raku Programming Language code to create ASTs.

Prerequisites for RakuAST
-------------------------
It is the intent to have all Raku source code be parsed by the (new) Raku grammar (and associated Actions) in the future, just as it is now with the legacy grammar.  Since the Raku core setting (which contains the Raku code for most of the implementation of the Raku Programming Language) must also be parsed by this, it implies that the RakuAST classes must exist **before** there is any Raku Programming Language.

This is a chicken-and-egg problem that is solved in Rakudo by the so-called "bootstrap".  This is quite a sizeable chunk of NQP code that "manually" creates enough functionality to allow the Raku core setting to build itself up to a fully functional implementation of the Raku Programming Language.

When *Jonathan Worthington* started the RakuAST project, they didn't want to have to implement all of that functionality in NQP yet again.  So they devised a neat hack by creating a rather simple parser that would read Raku-like code, and convert that to NQP source code that would create all of the **360+** RakuAST classes when run.  Of course, that Raku-like code still does not have the full Raku capabilities, but it does make the implementation task a lot easier than it would have been if it had be all written in NQP.

Example of a RakuAST class' internals
-------------------------------------
Let's have look a simple example of such a RakuAST class.  For instance, the `RakuAST::StrLiteral` class that we've seen earlier in the [first instalment](https://dev.to/lizmat/rakuast-for-early-adopters-576n).
```
class RakuAST::StrLiteral is RakuAST::Literal {
    has Str $.value;

    method new(Str $value) {
        my $obj := nqp::create(self);
        nqp::bindattr($obj, RakuAST::StrLiteral, '$!value', $value);
        $obj
    }

    # other methods

    method IMPL-EXPR-QAST(RakuAST::IMPL::QASTContext $context) {
        my $value := $!value;
        $context.ensure-sc($value);
        my $wval := QAST::WVal.new( :$value );
        QAST::Want.new($wval,'Ss', QAST::SVal.new(:value(nqp::unbox_s($value))))
    }
}
```
For simplicity's sake, only two methods are shown.  The `new` method, taking a single positional `Str $value`.  Which needs some NQP to create the object and bind the value to the `$!value` attribute.

And we see an `IMPL-EXPR-QAST` method.  That method will be called whenever that RakuAST object needs to generate QAST (a pre-cursor to actual bytecode) for itself.

If you find this very interesting, you probably want to read the RakuAST [README](https://github.com/rakudo/rakudo/blob/main/src/Raku/ast/README.md#rakuast).  And the actual source code of the RakuAST classes can be found [in the same directory](https://github.com/rakudo/rakudo/tree/main/src/Raku/ast).  And if you're really feeling adventurous and you have the [Rakudo repository](https://github.com/rakudo/rakudo) checked out, you can have a look at the generated NQP code in `gen/moar/ast.nqp`.

What's the point?
-----------------
So why am I even mentioning this?  Because the `RakuAST` classes *look* like they're actual Raku classes, but they're really NQP subroutines wrapped up to appear like Raku classes.  Which results in unexpected failure modes if there's some error in your calls to `RakuAST` classes.   In other words: the edges are a little bit sharper with RakUAST classes, and [LTA](https://docs.raku.org/language/glossary#LTA) error messages can and will happen.  It's one of the "benefits" of living on the edge!

Of course, as a user of RakuAST classes, you should only be interested in the `new` method, and any other non-internal methods.  Sadly, it is way *too early* in the bootstrap to mark internal methods with an [`is implementation-detail`](https://docs.raku.org/language/traits#is_implementation-detail_trait) trait, so another heuristic is needed.  And that would be: "consider any ALL-CAPS methods to be off-limits".

Conclusion
----------
This instalment gives an actual example of how you can use RakuAST in your code today.  And gives some technical background on the implementation of RakuAST classes, which still is a little sharp around the edges.

The intended audience are those people willing to be early adopters of these exciting new features in the Raku Programming Language.
