# Quicker to assume

The Raku Programming Language has a nice feature that goes by different names: ["currying"](https://en.wikipedia.org/wiki/Currying), ["priming"](https://docs.raku.org/syntax/priming) or ["partial application"](https://en.wikipedia.org/wiki/Partial_application).  Oddly enough, the method name associated with the feature is called [`assuming`](https://docs.raku.org/routine/assuming).

## Some examples

So what does the `.assuming` method do?  In short, it creates a subroutine from an existing subroutine (or method, or pointy block) with one or more arguments already "filled in".  For instance:
```
sub hello($firstname, $lastname) {
    say "Hello, $firstname $lastname";
}
hello "Joe", "Smith";   # Hello, Joe Smith

my &hi-smith = &hello.assuming(*, 'Smith');  # fill in 2nd positional
hi-smith("Joe");  # Hello, Joe Smith
```
This is a simple example in which one positional parameter is replaced.  Replacing named arguments is also possible, and adding values to slurpy arrays as well.

Let's take the [`max`](https://docs.raku.org/routine/max#(Operators)\_infix_max) subroutine that returns the maximum of the given values.  In this case we make sure that the maximum value of any number of values given, is at least 0.  By making sure that the value `0` is always added to any list of values specified.  Which also handles the case if called without any arguments.
```
my &max0 = &max.assuming(0);  # always add 0 as a value to be checked
say max  -1, -2, -3;  # -1
say max0 -1, -2, -3;  # 0, because 0 is greater than -1;

say max;   # -Inf, smallest possible numeric value
say max0;  # 0, because max(0) is 0
```

## Original implementation

The original implementation of `.assuming` was done by *Brian S. Julin* in July 2015.  The commit message mentioned:

> This currently uses EVAL to construct the closure, which is LTA, but it gives us something functional/testable to work forward from.

In the close to 10 years since then, only small tweaks and fixes have been applied by a group of Rakudo core developers.  But the essentially hacky approach of building a string with Raku code, and then [`EVAL`](https://docs.raku.org/routine/EVAL) it, did not change.  Simply because there were not any alternative solutions.

Apart from being hacky, the `EVAL` approach added quite a bit of *runtime* overhead.  Not only would it take time to create the string with Raku code to be evaluated, and run the evaluation itself (which could potentially happen at compile time, so not a real worry for modules).  It would also process arguments at runtime.  That runtime overhead also caused an issue to be made in early 2019: [.assuming is painfully slow](https://github.com/rakudo/rakudo/issues/2599).  But since there was no alternative approach available, the issue remained dormant for more than 6 years.

## RakuAST

Fastforward to today.  There is now an alternative way of building code that is to be executed: [`RakuAST`](https://docs.raku.org/type/RakuAST).

The [RakuAST project](https://news.perlfoundation.org/post/gp_rakuast) was started by *Jonathan Worthington*.  From the grant proposal:

> The goal of RakuAST is to provide an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) that is part of the Raku language specification, and thus can be relied upon by the language user. Such an AST is a prerequisite for a useful implementation of macros that actually solve practical problems, but also offers further powerful opportunities for the module developer.

About two years ago yours truly blogged a little about it already: [RakuAST for Early Adopters](https://dev.to/lizmat/rakuast-for-early-adopters-576n).  Since then, development has continued to a point where almost all features of Raku can be expressed using the new Raku grammar, which builds its AST using RakuAST (now at 94.9% of test files completely passing).

Having looked at making `.assuming` produce faster code in 2019 already, it felt like a good time to do another attempt.  But this time using RakuAST!

## Prototyping

The goal would be that for a simple case like:
```
sub foo(Int $a) { ... }
my &bar = &foo.assuming(42)
```
`.assuming` would effectively create a subroutine `bar` that would just be:
```
sub bar() { foo(42) }
```
without any additional runtime overhead.

Just over two weeks ago a prototype was created.  Outside of the core setting, just in a simple script.  And the initial results turned out to be promising: some simple examples turned out to be already 40x as fast as the original implementation.  As more features were added, that number went down for some cases as was to be expected.

As for the "more features" bit:  wow, quite a number of features that you can have in the signature of a subroutine!

## Nooks and crannies

Or how I learned more about signatures than I would ever like to know.  Apart from positional and named arguments, you can also have slurpy arrays (3 types) and slurpy hashes.  And captures (named or nameless), without or without sub-signatures.  And then you can specify a *variable* as a value, which can be changed by the subroutine if it was defined with `is raw`.  Various constraints can be applicable: as code blocks, or as types: possibly parameterized, and/or coerced and/or with a type smiley.  And then there are generics (aka type object placeholders).  Put them all together in a mix and let the fun begin!

That was actually the most work: getting all of the interactions right.  And while doing that, 3 bugs were found that had to do with the use of generics in parameterization, and role composition.  One of them is fortunately already fixed.

In any case, as of this writing it appears that every possible combination of signature features are supported by the new implementation of `.assuming`.

## Introspection

In Raku one can easily introspect aspects of code: given a `Sub` object, one can ask for its signature (which is a [`Signature`](https://docs.raku.org/type/Signature) object).  And given a `Signature` object, one can ask for its parameters (which would be [`Parameter`](https://docs.raku.org/type/Parameter) objects), and one can ask for the return value / type constraint (which would generally be a [type object](https://docs.raku.org/syntax/Type%20Objects).  And given a `Parameter` object, one can ask whether it is a named or positional parameter, whether it optional or not, its constraint and many other aspects.

To be able to build a new `Sub` object, these aspects of signatures, parameters and types need to be converted to `RakuAST` objects.  Any code trying to do similar things as `.assuming`, could benefit from this logic.  Therefore this part of the work on `.assuming` has also been published as the [`RakuAST::Utils`](https://raku.land/zef:lizmat/RakuAST::Utils) distribution.

To give you an idea how this looks like:
```
use RakuAST::Utils;
sub foo(Int $a, Int(Str) $b --> Str:D) { "foo" }
say SignatureAST(&foo.signature);
```
    RakuAST::Signature.new(
      parameters => (
        RakuAST::Parameter.new(
          type   => RakuAST::Type::Simple.new(
            RakuAST::Name.from-identifier("Int")
          ),
          target => RakuAST::ParameterTarget::Var.new(
            name => "\$a"
          )
        ),
        RakuAST::Parameter.new(
          type   => RakuAST::Type::Coercion.new(
            base-type  => RakuAST::Type::Simple.new(
              RakuAST::Name.from-identifier("Int")
            ),
            constraint => RakuAST::Type::Simple.new(
              RakuAST::Name.from-identifier("Str")
            )
          ),
          target => RakuAST::ParameterTarget::Var.new(
            name => "\$b"
          )
        ),
      ),
      returns    => RakuAST::Type::Definedness.new(
        base-type => RakuAST::Type::Simple.new(
          RakuAST::Name.from-identifier("Str")
        ),
        definite  => True
      )
    )

The functionality offered by this distribution could possibly become core at some point, but the interface / API should mature a bit before that.  Also, the RakuAST interface itself isn't fully stable yet, so this module can provide a more stable interface as a central place to handle any RakuAST interface changes.

## Testing

There were quite a few tests for the `.assuming` logic in the [official Raku test suite (aka "roast")](https://github.com/raku/roast?tab=readme-ov-file#the-official-raku-test-suite).  Alas, many of them were testing implementation details of the signatures created by the old (incomplete) implementation.  They have either been adjusted, or removed altogether.  And some new tests have been added, to cover some features that were not supported yet.

## Performance

Benchmarking during the development process showed that the `RakuAST` approach to `.assuming` was between 2.5x and 50x faster than the old string `EVAL`.  However, the slowdown that was mentioned in the [issue](https://github.com/rakudo/rakudo/issues/2599) turned out to be underestimated: quite a few tests showed an actual slowdown of up to 80x.  So even being 50x faster on a 80x slower case, is still significantly slower  :-(

A small benchmark on a 2020 M1 Apple Silicon processor (no JIT available):
```
sub a($a) { $a }
sub b() { a(42) }
BEGIN my &c = &a.assuming(42);
my &d = &a.assuming(42);

{ a(42) for ^10_000_000; say now - ENTER now }  # 0.352358404
{ b()   for ^10_000_000; say now - ENTER now }  # 0.415636707
{ c()   for ^10_000_000; say now - ENTER now }  # 1.234486065
{ d()   for ^10_000_000; say now - ENTER now }  # 1.404967796
```
Which shows that the new `.assuming` approach (`c`) is still almost 3x as slow (1.234486065 / 0.415636707 = 2.97) as a handmade intermediate subroutine (`b`).

Also note that running `.assuming` at runtime makes it even slower still (3.38x): probably because it then misses some static optimizations that are run at compile time.

With more work done on RakuAST, specifically with regards to optimizations, it should be possible to eliminate the difference between `b()` and `c()`.

On the other hand, please keep in perspective that these benchmarks were run for **10 million times**, where the runtime was mostly spent in calling a subroutine, and not much else.  In any fleshed out subroutine, the `.assuming` overhead will probably be drowned away in the time needed to execute the rest of the code in the subroutine.

## Conclusion

The RakuAST project has reached such a maturity that it could be used to re-imagine the currying / priming / partial applocation logic of the Raku Programming Language.  And that this re-imagination has made that functionality an order of magnitude faster, while still not being fully optimized yet.

Thanks to the [Raku Foundation](https://raku.foundation) for supporting this work on `.assuming`.

*If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!*
