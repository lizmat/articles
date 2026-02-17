# Method Not Found Fallback

> This is part eleven in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the interface method that can be provided in a class to handle calls to methods that do not actually exist in this class (or its parents).

## Some background

Method dispatch in the [Raku Programming Language](https://raku.org) is quite complicated, especially in the case of multi-dispatch.  But the first step is really to see if there is **any** (multi) method with the given name.

Internally this is handled by the [`find_method` method](https://docs.raku.org/type/Metamodel/MROBasedMethodDispatch#method_find_method) on the meta-object of the class (which is typically called with the [`.^method` syntax](https://docs.raku.org/language/operators#methodop_%2E^) on the class).

So for example:
```raku
# an empty class
class A { }

dd A.^find_method("foobar");  # Mu
dd A.^find_method("gist");    # proto method gist (Mu $:: |) {*}
```
Note that an empty class `A` (which by default inherits from the [`Any` class](https://docs.raku.org/type/Any)) does **not** provide a method "foobar" (indicated by the [`Mu` type object](https://docs.raku.org/type/Mu)).  But it *does* provide a `gist` method (indicated by the `proto method gist (Mu $:: |) {*}` representation of a [`Callable`](https://docs.raku.org/type/Callable)) because that is inherited from the `Any` class..

It's this logic that is internally used by the dispatch logic to link a method name to an actual piece of code to be executed.

## Method not found

If the dispatch logic can not find a method by the given name, then it will throw an [`X::Method::NotFound`](https://docs.raku.org/type/X/Method/NotFound) error.  One could of course use a [`CATCH`](https://docs.raku.org/language/exceptions#Catching_exceptions) phaser to handle such cases (as seen in [part 6](https://dev.to/lizmat/catch-control-i9o) of this series).

But there's a better and easier way to handle method names that could not be found.  That is, if you're interested in somehow making them less fatal.

## FALLBACK

If a class provides a [`FALLBACK` method](https://docs.raku.org/language/typesystem#Fallback_method) (either directly in its class, or by one of its base classes, or by ingestion of a role), then that method will be called whenever a method could not be found.  The name of the method will be passed as the first argument, and all other arguments will be passed verbatim.

A contrived example in which a non-existing method returns the *name* of the method, but only if there were **no** arguments passed:
```raku
class B {
    method FALLBACK($name) { $name }
}
say B.foo;      # foo
say B.bar(42);  # Too many positionals passed; expected 2 arguments but got 3
```

So is there something special about the `FALLBACK` method?   No, its only specialty is *when* it is being called.  So you can make it a [`multi` method](https://docs.raku.org/syntax/multi-method):
```raku
class C {
    multi method FALLBACK($name) {
        $name
    }
    multi method FALLBACK($name, $value) {
        "$value.raku() passed to '$name'"
    }
}
say C.foo;      # foo
say C.bar(42);  # 42 passed to 'bar'
```
On the other hand if you don't care about any arguments at all and just want to return [`Nil`](https://docs.raku.org/type/Nil), you can specify the nameless capture `|` as the signature:
```raku
class D {
    method FALLBACK(|) { Nil }
}
say D.foo;      # Nil
say D.bar(42);  # Nil
```

## An actual application

The Raku Programming Language allows hyphens in identifier names (usually referred to as ["kebab case"](https://en.wikipedia.org/wiki/Letter_case#Kebab_case).  Many programmers coming from other programming languages are not really used to this: they are more comfortable with using underscores in identifiers (["Snake case"](https://en.wikipedia.org/wiki/Letter_case#Snake_case)).

Modern Raku programs usually use kebab case in identifiers.  So it's quite common for programmers to make the mistake of using an underscore where they should have been using a hyphen.  If such an error is made when writing a program, it will be a runtime error.  Which can be annoying.  In such a case, a `FALLBACK` method can be a useful thing to have if it could correct such mistakes.

This could look something like this:
```raku
class E {
    method foo-bar() { "foobar" }

    method FALLBACK($name, |c) {
        my $corrected = $name.trans("_" => "-");
        if self.^find_method($corrected, :no_fallback) -> &code {
            code(self, |c)
        }
        else {
            X::Method::NotFound.new(
              :invocant(self), :method($name), :typename(self.^name)
            ).throw;
        }
    }
}
```
The "E" class has a method "foo-bar".  And a method "FALLBACK" that takes the name of the method that was not found (and putting any additional arguments in the [`Capture`](https://docs.raku.org/type/Capture) "c").

> Note that the "c" is just an idiom for the name of a capture.  It could have any name, but "c" is nice and short.  If you want to use a name that's more clear to you, then please do so.  As long as you use the same name later on.

It then converts all occurrences of underscore to hyphen in the name and then tries to find a `Callable` for that name.  If that is successful it will execute the code with any arguments that were given by flattening the [`|c` capture](https://docs.raku.org/language/signatures#Capture_parameters).  Otherwise it throws a "method not found" error.

> The `:no_fallback` argument is needed to prevent the method lookup from producing the "FALLBACK" method if the method was not found.  Otherwise the code would loop forever.

So now this code will work instead of causing an execution error.
```raku
say E.foo-bar;  # foobar
say E.foo_bar;  # foobar
```
To make this more generally usable this code could be put into a [`role`](https://docs.raku.org/language/objects#Parameterized_roles) and have that live as an installable module.  But that would be extra work, wouldn't it?

Fortunately the writing of this blog post initiated the development of such a role (and associated distribution) called [`Method::Misspelt`](https://raku.land/zef:lizmat/Method::Misspelt).  How's that for BDD (Blog Driven Development)?

> Note that the module introduces a few more features and optimizations, while handling keeping track of multiple classes ingesting the same role.  See the [internal documentation](https://github.com/lizmat/Method-Misspelt/blob/main/lib/Method/Misspelt.rakumod) for more information.

## Conclusion

This concludes the eleventh episode of cases of UPPER language elements in the Raku Programming Language, the fourth episode discussing interface methods.

In this episode the `FALLBACK` method was described, as well as some simple customizations.  And a [bonus module](https://raku.land/zef:lizmat/Method::Misspelt) created for this blog post only.

Stay tuned for the next episode!
