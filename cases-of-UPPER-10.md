# Associative Methods

> This is part ten in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the interface methods that one can implement to provide a custom [`Associative`](https://docs.raku.org/type/Associative) interface in the [Raku Programming Language](https://raku.org).  Associative access is indicated by the [postcircumfix { } operator](https://docs.raku.org/language/operators#postcircumfix_{_}) (aka the "hash indexing operator").

> In a way this blog post is a [cat license](https://en.wikipedia.org/wiki/Fish_Licence) ("a dog licence with the word 'dog' crossed out and 'cat' written in crayon") from the [previous post](https://dev.to/lizmat/positional-methods-439i).  But there are some subtle differences between the `Positional` and the `Associative` roles, so just changing all occurrences of "POS" in "KEY" will not cut it.

## Some background

The `Associative` role is really just a marker, just as the `Positional` role.  It does **not** enforce any methods to be provided by the consuming class.  So why use it?  Because it is *that* constraint that is being checked for any variable with a `%` [sigil](https://docs.raku.org/language/glossary#Sigil).  An example:
```raku
class Foo { }
my %h := Foo;
```
will show an error:
```
Type check failed in binding; expected Associative but got Foo (Foo)
```
However, if we make that class ingest the `Associative` marker role, it works:
```raku
class Foo does Associative { }
my %h := Foo;
say %h<bar>;  # (Any)
```
and it is even possible to call the postcircumfix `{ }` operator on it!  Although it doesn't return anything really useful.

> Note that the binding operator `:=` was used: otherwise it would be interpreted as initializing a hash `%h` with a single value, which would complain with an "Odd number of elements found where hash initializer expected: Only saw: type object 'Foo'" execution error.

## Postcircumfix { }

The `postcircumfix { }` operator performs all of the work of slicing and dicing objects that perform the `Associative` role, and handling all of the adverbs: `:exists`, `:delete`, `:p`, `:kv`, `:k`, and `:v`.  But it is completely agnostic about how this is actually done, because all it does is calling the interface methods that are (implicitely) provided by the object, just as with the `Positional` role.  Except the names of the interface methods are different.  For instance:
```raku
say %h<bar>;
```
is actually doing:
```raku
say %h.AT-KEY("bar");
```
under the hood.  So these interface methods are the ones that actually know how to work on an [`Hash`](https://docs.raku.org/type/Hash), [`Map`](https://docs.raku.org/type/Map) or a [`PseudoStash`](https://docs.raku.org/type/PseudoStash), or any other class that does the `Associative` role.

## The Interface Methods

Let's introduce the cast of *this* show (the interface methods associated with the `Associative` role):

- [`AT-KEY`](https://docs.raku.org/language/subscripts#method_AT-KEY)
- [`EXISTS-KEY`](https://docs.raku.org/language/subscripts#method_EXISTS-KEY)
- [`DELETE-KEY`](https://docs.raku.org/language/subscripts#method_DELETE-KEY)
- [`ASSIGN-KEY`](https://docs.raku.org/language/subscripts#method_ASSIGN-KEY)
- [`BIND-KEY`](https://docs.raku.org/language/subscripts#method_BIND-KEY)
- [`STORE`](https://docs.raku.org/type/Associative#method_STORE)
- [`keys`](https://docs.raku.org/type/Map#method_keys)

### AT-KEY

The `AT-KEY` method is the most important method of the interface methods of the `Associative` role: it is expected to take the argument as a key, and return the value associated with that key.

> Note that the key does **not** need to be a string, it could be any object.  It's just that the implenentation of `Hash` and `Map` will coerce the key to a string.

The `AT-KEY` method should return a container if that is appropriate, which is usually the case.  Which means you probably should specify [`is raw`](https://docs.raku.org/routine/is%20raw) on the `AT-KEY` method if you're implementing that yourself.
```raku
say %h<bar>;  # same as %h.AT-KEY("bar")
```

### EXISTS-KEY

The `EXISTS-KEY` method is expected to take the argument as a key, and return a [`Bool`](https://docs.raku.org/type/Bool) value indicating whether that key is considered to be existing or not.  This is what is being called when the [`:exists`](https://docs.raku.org/type/Hash#:exists) adverb is specified.
```raku
say %h<bar>:exists;  # same as %h.EXISTS-KEY("bar")
```

### DELETE-KEY

The `DELETE-KEY` method is supposed to act very much like the `AT-KEY` method.  But it is also expected to remove the key so that the `EXISTS-KEY` method will return `False` for that key in the future.  This is what is being called when the [`:delete`](https://docs.raku.org/type/Hash#:delete) adverb is specified.
```raku
say %h<bar>:delete;  # same as %h.DELETE-KEY("bar")
```

### ASSIGN-KEY

The `ASSIGN-KEY` method is a convenience method that *may* be called when assigning (`=`) a value to a key.  It takes 2 arguments: the key and the value.  It functionally defaults to `object.AT-KEY(key) = value`.  A typical reason for implementing this method is performance.
```raku
say %h<bar> = 42;  # same as @a.ASSIGN-KEY("bar", 42)
```

### BIND-KEY

The `BIND-KEY` method is a method that will be called when binding (`:=`) a value to a key.  It takes 2 arguments: the key and the value.  If not implemented, binding will always fail with an execution error.
```raku
say %h<bar> := 42;  # same as %h.BIND-KEY("bar", 42)
```

### STORE

The `STORE` method accepts the values to be (re-)initializing with as an [`Iterable`](https://docs.raku.org/type/Iterable) **and** returns the invocant ([`self`](https://docs.raku.org/syntax/self)).

The `:INITIALIZE` named argument will be passed with a `True` value if this is the first time the values are to be set.  This is important if your data structure is supposed to be immutable: if that argument is `False` or not specified, it means a re-initialization is being attempted.
```raku
say %h = a => 42, b => 666;  # same as %h.STORE( (a => 42, b => 666) )
```

### keys

Although not an uppercase named method, it is an important interface method: the [`keys`](https://docs.raku.org/type/Map#method_keys) method is expected to return the keys in the object.
```raku
my %h = a => 42, b => 666;
say %h.keys;  # (a b)
```

## Handling simple customizations

Again, that's a lot to take in (especially if you didn't read the previous post)!  But if it is just a simple customization you wish to do to the basic functionality of e.g. a hash, you can simply inherit from the `Hash` class.  Here's a simple, contrived example that will return any values doubled as string:
```raku
class Hash::Twice is Hash {
    method AT-KEY($key) { callsame() x 2 }
}
my %h is Hash::Twice = a => 42, b => 666;
say "$_: %h{$_}" for %h.keys;
```
will show:
```
a: 4242
b: 666666
```
Note that in this case the [`callsame`](https://docs.raku.org/syntax/callsame) function is called to get the actual value from the array.

## Helpful modules

If you want to be able to do more than just simple modifications, but for instance have an existing data structure on which you want to provide an `Associative` interface, it becomes a bit more complicated and potentially cumbersome.

Fortunately there are a number of modules in the ecosystem that will help you to create a consistent `Associative` interface for your class.

### Hash::Agnostic

The [`Hash::Agnostic`](https://raku.land/zef:lizmat/Hash::Agnostic) distribution provides a `Hash::Agnostic` role with all of the necessary logic for making your object act as a `Hash`.  The only methods that *must* be provided, are the `AT-KEY` and `keys` methods.  Your class *may* provide more methods for functionality or performance reasons.

### Map::Agnostic

The [`Map::Agnostic`](https://raku.land/zef:lizmat/Map::Agnostic) distribution provides a `Map::Agnostic` role with all of the necessary logic for making your object act as a `Map`.  The only methods that you *must* be provided, are the `AT-KEY` and `keys` methods.  As with `Hash::Agnostic`, your class *may* provide more methods for functionality or performance reasons.

### Example class

A very contrived example in which a `Hash::Int` class is created that provides an `Associative` interface in which the keys can only be integer values.  Under the hood it uses an array to store values:
```raku
use Hash::Agnostic;

class Hash::Int does Hash::Agnostic {
    has @!values;
    method AT-KEY(Int:D $index) {
        @!values.AT-POS($index)
    }
    method keys() {
        (^@!values).grep: { @!values.EXISTS-POS($_) }
    }
    method STORE(\values) {
        my @values;
        for Map.CREATE.STORE(values, :INITIALIZE) {
            @values.ASSIGN-POS(.key.Int, .value)
        }
        @!values := @values;
    }
}
```
and can be used like this:
```raku
my %h is Hash::Int = <42 a 666 b 137 c>;
say %h<42>;
dd %h;
```
which would show:
```
a
Hash::Int.new(42 => "a",137 => "c",666 => "b")
```
Again, note that the `STORE` method had to be provided by the class to allow for the `is Hash::Int` syntax to work.

> If you're wondering what's happening with `Map.CREATE.STORE(values, :INITIALIZE)`: the initialization logic of hashes and maps allows for both separate key,value initialization, as well as key=>value initialization.  And any mix of them.  So this is just a quick way to use that rather specialized logic of `Map.new` to create a consistent [`Seq`](https://docs.raku.org/type/Seq) of [`Pair`](https://docs.raku.org/type/Pair)s with which to initialize the underlying array.

> Raku actually has a syntax for creating a `Hash` that only takes `Int` values as keys: `my %h{Int}`.  This creates a so-called ["object hash"](https://docs.raku.org/language/hashmap#Non-string_keys_(object_hash)) with different performance characteristics to the approach taken with `Hash::Int`.

## Conclusion

This concludes the tenth episode of cases of UPPER language elements in the Raku Programming Language, the third episode discussing interface methods.

In this episode the `AT-KEY` family of methods were described, as well as some simple customizations and  some handy Raku modules that can help you create a fully functional interface: `Map::Agnostic` and `Hash::Agnostic`.

Stay tuned for the next episode!
