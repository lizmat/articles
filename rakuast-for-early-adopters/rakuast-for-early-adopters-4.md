Walking the RakuAST Tree
========================
RakuAST for Early Adopters (part 4)

One of the interesting things about ASTs in general, and RakuAST in particular, is that you can walk the tree to look for certain objects, or combination of objects, and act accordingly.

To allow walking the tree, each `RakuAST::Node` object has a `.visit-children` method that takes a `Callable` that will be executed for all of the applicable "children" of the invocant.  So what are "children" in this context?  Let's take an example we've seen before:
```
RakuAST::ApplyInfix.new(
  left  => RakuAST::IntLiteral.new(42),
  infix => RakuAST::Infix.new("+"),
  right => RakuAST::IntLiteral.new(666)
)
```
In this example, the "left" (`RakuAST::IntLiteral.new(42)`), "infix" (`RakuAST::Infix.new("+")`) and "right" (`RakuAST::IntLiteral.new(666)`) are considered to be "children" of the `RakuAST::ApplyInfix` object.  In this case, these "children" do not have children of their own.  But in many cases, they do: in which case, the `.visit-children` will be called on these objects as well.

Now, this sounds rather complicated.  But we can make it a lot less complicated by wrapping the complexity into a subroutine.

Grepping the tree
-----------------
So let's make a `grep` subroutine that takes a `RakuAST::Node` object and a matcher for the object, that will visit all of its "children" recursively.  And which returns a [`Seq`](https://docs.raku.org/type/Seq) of the `RakuAST::Node` objects that matched.
```
sub grep(RakuAST::Node:D $ast, $matcher) {
    sub visitor($ast) {                   # recursive visitor
        take $ast if $ast ~~ $matcher;    # accept if matched
        $ast.visit-children(&?ROUTINE);   # visit its children
    }

    gather $ast.visit-children(&visitor)  # gather the takes
}
```
This is an excellent situation to use the [`gather`and `take`](https://docs.raku.org/language/control#gather/take) functionality of the Raku Programming Language.

The `gather` returns a `Seq` and a dynamic scope in which each `take` will *lazily* be produced as an element in that `Seq`.  Its argument is an expression that will be executed within that dynamic scope: this can be a [`Block`](https://docs.raku.org/type/Block), but it doesn't have to be.

The `visitor` subroutine takes a `RakuAST::Node` object as its argument, and checks that with the given `$matcher`, which is lexically visible to this subroutine.  And then visits all of its children, with a call to itself (which is what [`&?ROUTINE`](https://docs.raku.org/language/variables#&%3FROUTINE) allows you to do).

Finding the tree
----------------
The `RakuAST` tree of a program is only available in any [`CHECK`](https://docs.raku.org/language/phasers#CHECK) phaser that a program has, in the form of the `$*CU` dynamic variable (with "CU" being short for "CompUnit").  So let's look at the RakuAST tree of the most trivial program:
```
CHECK { say $*CU }
```
which will output:
```
RakuAST::CompUnit.new(
  statement-list => RakuAST::StatementList.new(
    RakuAST::Statement::Expression.new(
      expression => RakuAST::StatementPrefix::Phaser::Check.new(
        RakuAST::Block.new(
          body => RakuAST::Blockoid.new(
            RakuAST::StatementList.new(
              RakuAST::Statement::Expression.new(
                expression => RakuAST::Call::Name.new(
                  name => RakuAST::Name.from-identifier("say"),
                  args => RakuAST::ArgList.new(
                    RakuAST::Var::Dynamic.new(
                      "\$*CU"
                    )
                  )
                )
              )
            )
          )
        )
      )
    )
  ),
  comp-unit-name => "E185D65E1AF12CAC5CCD46AB4C1AF7A3FF7089B7",
  setting-name   => "CORE.d"
)
```
As you can see, there's quite a lot there already.  So it's important that we can navigate through it easily.  Let's take our `grep` routine for a spin:

Picking local fruit
-------------------
In this example, we will collect all of the `=data` Rakudoc blocks from the source code, extract the text from that, and store that in a `@data` array, and show the contents of that array:
```
my @data = CHECK {
    grep($*CU, { $_ ~~ RakuAST::Doc::Block && .type eq 'data' })
      .map(*.paragraphs.join(' ').trim-trailing)
}

=head1 Victory

=data Blue
=data Yellow

say @data;  # [Blue Yellow]
```
The `grep` routine that we created earlier, is called with the compilation unit (`$*CU`) and a code block.  That code block first checks whether the given object is a `RakuAST::Doc::Block` object (`$_ ~~ RakuAST::Doc::Block`), and if it is, whether the type is equal to 'data' (`.type eq 'data'`).

Then the resulting values are mapped to their text content (`.map()`), by concatenating the paragraphs with a space between them (`.paragraphs.join(' ')` and then removing any trailing whitespace (`.trim-trailing`).  The latter is done, because all of the newlines until the next code or rakudoc block *are* preserved, and we're not interested in that in this case.

Note that the `CHECK` phaser returns the text from the selected `RakuAST` objects, and stores that in the `@data` array.  Many [phasers](https://docs.raku.org/language/phasers) in Raku return the value of the final expression, which is a handy feature to have!

Oh, and by the way, in this example we've almost created the [`$=data` feature](https://design.raku.org/S26.html#Data_blocks) that wasn't implemented in Raku yet (so far).

Picking foreign fruit
---------------------
Now, this is all nice and good.  But what if you want to pick out elements from *another* source-file?  The `grep` routine doesn't care where the `RakuAST` object came from.  So, if you want to obtain the `=data` blocks from another file, the only thing you need to do is to create a `RakuAST` object of that file.  And that's where the `.AST` method comes in again:
```
my @data = grep(
  $filename.IO.slurp.AST,
  { $_ ~~ RakuAST::Doc::Block && .type eq 'data' })
    .map(*.paragraphs.join(' ').trim-trailing)
);
```
In other words: given a `$filename` as a string. turn that into an [`IO::Path`](https://docs.raku.org/type/IO/Path) with `.IO`.  Then read all of the contents of the file into a string (`.slurp`) and then create a RakuAST tree out of it with `.AST`.  And then `grep` and `.map` as before.

Conclusion
----------
This installment introduces the `$*CU` dynamic variable in `CHECK` phasers and shows how you can extract `RakuAST` objects from a RakuAST tree using the `.visit-children` method on `RakuAST` objects.

The intended audience are those people willing to be early adopters of these exciting new features in the Raku Programming Language.
