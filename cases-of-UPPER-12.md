# Store Proxy Fetch

> This is part twelve in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the various aspects of containers in the [Raku Programming Language](https://raku.org).

## Containers and binding

This may come as a shock to some, but Raku really *only* knows about **binding** (`:=`) values.  Assignment is [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar).

What?

Most of the scalar variables that you see in Raku, are really objects that have an attribute to which the value is *bound* when you *assign* to the variable.  These objects are called [`Scalar`](https://docs.raku.org/type/Scalar) containers, or just short ["containers"](https://docs.raku.org/language/containers).

> In short: assignment in Raku is binding to an attribute in a `Scalar` object.  [Grokking](https://en.wikipedia.org/wiki/Grok#In_computer_programmer_culture) how containers work in Raku (as opposed to many other programming languages) is one of the rites of passage to becoming an experienced and efficient developer in Raku.  It sure was a rite of passage for yours truly!

In a simplified representation (almost [pseudocode](https://en.wikipedia.org/wiki/Pseudocode)) one can think of these `Scalar` objects like this:
```raku
class Scalar {
    has $!value;
    method STORE($value) {
        $!value := $value
    }
    method FETCH() {
        $!value
    }
}
```
And an assignment such as:
```raku
my $a;
$a = 42;
```
can be thought of as:
```raku
my $a := Scalar.new;
$a.STORE(42);
```
and showing the value of a variable (as in `say $a`), can be thought of as:
```raku
say $a.FETCH;
```
In reality it's of course slightly more complicated, because there are such things as type constraints that can exist on a variable (such as `my Int $a`).  And variables may have a default value (as in `my $a is default(42)`.

> This type of information is kept in a so-called "container descriptor", which is considered to be an [implementation detail](https://docs.raku.org/syntax/is%20implementation-detai) (as in: don't depend on its functionality directly, the interface to it might change at any time).

Many shortcuts are made in implementing this, in the interest of performance.  Because assignments are one of the very basic operations in a programming language.  And you want those to be fast!

As a mental model, this is very useful for understanding quite a few constructs in Raku.

## Return values

In Raku any value returned at the end of a block (or with the [`return`](https://docs.raku.org/syntax/return) statement) are de-containerized by default.  This means that something like this:
```raku
my $a = 42;
sub foo() { $a }
foo() = 666;  # Cannot modify an immutable Int (42)
```
will not work.  What if you could return the container from a block?  Then you could just assign to it!  There are indeed two easy ways to return something with the container intact: the [`is rw`](https://docs.raku.org/routine/is%20rw) trait:
```raku
my $a = 42;
sub foo() is rw { $a }
foo() = 666;
say $a;  # 666
```
Alternately, one can use the [`return-rw`](https://docs.raku.org/syntax/return-rw) statement.
```raku
my $a = 42;
sub foo() { return-rw $a }
foo() = 666;
say $a;  # 666
```
This feature is for instance used if you want to assign to an array element.  In [part 9](https://dev.to/lizmat/positional-methods-439i) of this series it was shown that the `AT-POS` method is what is being called postcircumfix `[ ]`.  In the core, both the postcircumfix `[ ]` operator as well as the underlying `AT-POS` method have this trait set for performance (as the `return-rw` statement has slightly more overhead).

## Binding to containers

If an array is initialized with values, it will create containers for each element and put the right value in the right place.  So you cannot only *fetch* the values from there, but you can also *assign* to these containers.  And it is also possible to *bind* such a container to another variable.

Binding to containers allows for features in Raku that are considered "magic" by some, and maybe "too magic" by others.  They are part of common idioms in the Raku Programming Language.

For instance: incrementing all values in an array:
```raku
my @a = 1,2,3,4,5;
$_++ for @a;
say @a;  # [2 3 4 5 6]
```
The topic variable `$_` **binds** to the elements of the array in a `for` loop.  So in each iteration it actually represents the container at that location and can thus be incremented.

> People coming from other languages may think that `$_` is a "reference" (or "pointer") to the actual memory location of the element in the array.  This notion is incorrect in Raku.  The topic variable `$_` is bound to the **container** of each element.  The container object itself is completely agnostic as to where it lives.  It's just a simple object that knows how to `FETCH` and `STORE`.

Slices to arrays also return containers.  For instance to increment only the first and last element in an array without knowing its size:
```raku
my @a = 1,2,3,4,5;
$_++ for @a[0, *-1];
say @a;  # [2 2 3 4 6]
```
The slice `[0, *-1]` produces 2 containers (one for the first element (`0`) and one for the last (`*-1`).  Only these containers get incremented, thus giving the expected result.

The same logic applies to the [`values`](https://docs.raku.org/routine/values) in hashes.
```raku
my %h = a => 42, b => 666;
$_++ for %h.values;
say %h;  # {a => 43, b => 667}
```
But binding containers can also be done directly in your code.  A contrived example:
```raku
my $a = 42;
my $b := $a;
$b = 666;
say $a;  # 666
```
Note that by assigning to `$b`, you're assiging to the container that lives in `$a`.  Because the binding of `$b` to `$a` effectively aliased the container in `$a`.

## Special containers

In Raku it is possible to assign to elements in an array that do not exist yet.
```raku
my @a;
@a[3] = 42;
say @a;  # [(Any) (Any) (Any) 42]
```
So there's no container yet for such an element.  Yet it is possible to assign to it!  How does that work?

The secret is really in the "container descriptor".  So let's refine our representation of the `Scalar` class:
```raku
class Scalar {
    has $!descriptor;
    has $!value;
    method STORE($value) {
        $!value := $!descriptor.process($value);
    }
    method FETCH() {
        $!value
    }
}
```
Note that storing a value has now become a little more involved (`$!descriptor.process($value)` rather than just `$!value`).  So the descriptor object's `process` method takes the value, does what it needs to do with it, and then returns it so that it can be bound to the attribute.

This descriptor object is typically responsible for the name, the default type, type checking and a default value of a container.  And any additional logic that is needed.

> There are many different types of container descriptor classes in the Rakudo implementation, all starting with the `ContainerDescriptor::` name.  And all are considered to be implementation detail.

For instance, when an `AT-POS` method is called on a non-existing element in an array, a container with a special type of descriptor is created that "knows" to which `Array` it belongs, and at which index it should stored.  The same is true for the descriptor of the container returned by `AT-KEY` (as seen in [part 10](https://dev.to/lizmat/associative-methods-2mcl)) which knows in which `Hash` it should store when assigned to, and what key should be used.

It's this special behaviour that allows arrays and hashes to really just [DWIM](https://docs.raku.org/syntax/DWIM).

## Action at a distance

These special descriptors of containers for arrays and hashes also introduce an action-at-a-distance feature that you may or may not like.
```raku
my @a;
my $b := @a[3];
say @a;  # []
$b = 42;
say @a;  # [(Any) (Any) (Any) 42]
```
Note that the element in the array was *not* initialized after the binding, but only *after* a value was assigned to `$b`.  This behaviour was specifically implemented this way to prevent accidental [auto-vivification](https://docs.raku.org/language/subscripts#Autovivification).  For example:
```raku
my %h;
say %h<a><b>:exists;  # False
say %h;               # {}
%h<a><b> = 42;
say %h;               # {a => {b => 42}}
```
So even though `%h<a`> is considered to be an `Associative` because of the `<b>`, the `:exists` test will **not** create a `Hash` in `%h<a>`.  Only *after* a value has been assigned does `%h<a>` and `%h<a><b>` actually spring to life.

## Proxy

This rather lengthy introduction / diversion was to make you aware of some of the underlying mechanics of containers.  Because Raku supplies a full customizable class that allows you to create your own container logic: [`Proxy`](https://docs.raku.org/type/Proxy).  And understanding what containers are about is helpful when working with that class.

Creation of such a container is quite easy: all you need to supply are a `method` for *fetching* the value, and a `method` for *storing* a value (very similar to the pseudocode representation at the start of this blog post).  This is done with the named arguments `FETCH` and `STORE`.  Creation of a `Proxy` object is usually done inside a subroutine for convenience.  A contrived example:
```raku
sub answer is rw {
    my $value = 42;
    Proxy.new(
      FETCH => method () {
          $value
      },
      STORE => method ($new) {
          say "storing $new";
          $value = $new;
      }
    )
}
my $a := answer;
say $a;    # 42
$a = 666;  # storing 666
say $a;    # 666
```
The careful reader will have noticed that the `is rw` attribute needs to be specified on the subroutine, otherwise the `Proxy` would be de-containerized on return.  And that the result of calling the `answer` subroutine was bound (`:=`) instead of assigned, because assignment would also cause de-containerization and thus completely defeat the purpose of this exercise.

Because the code in the supplied methods [closes over](https://docs.raku.org/syntax/closures) the lexical variable `$value`, that variable stays alive until the `Proxy` object is destroyed.  So it offers an easy way to actually store the value for this `Proxy` object.

Inside the supplied methods you are completely free to put whatever code that you want.  As an example how that could work in a module, the [`Hash::MutableKeys`](https://raku.land/zef:lizmat/Hash::MutableKeys) distribution was created.  Another case of BDD (Blog Driven Development)!

> You may have observed that the `Proxy` object does not allow for a descriptor.  It was not considered to be needed, as you have all the flexibility you could possibly want.  If you want one in your `Proxy` objects, you can create one yourself.

## Conclusion

This concludes the twelfth episode of cases of UPPER language elements in the Raku Programming Language, the fifth episode discussing interface methods.

In this episode containers were described, as well as the special `Proxy` class with its `STORE` and `FETCH` named arguments.  This `Proxy` class provides a fully customizable container implementation.

Stay tuned for the next episode!
