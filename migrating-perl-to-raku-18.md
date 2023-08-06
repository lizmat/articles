# Failure Is An Option
This blog post explains how failure is an option in the Raku Programming Language, how you can create your own exceptions, and how this all differs from Perl.

## Introducing Failure
In Perl, you need to prepare for a possible exception by using `eval` or some version of `try` when using a CPAN module. In Raku, you can do the same with `try` (as seen before).

But Raku also has another option: [`Failure`](https://docs.raku.org/type/Failure), which is a special class for wrapping an `Exception`. Whenever a `Failure` object is used in an unanticipated way, it will throw the `Exception` it is wrapping.

Here is a simple example:
```
# Raku
my $handle = open "non-existing file";
say "we tried to open the file";
say $handle.get;  # unanticipated use of $handle, throws
say "this will never be shown";
# Failed to open file non-existing file:
#   No such file or directory
```
The [`open`](https://docs.raku.org/type/independent-routines#sub_open) function in Raku returns an [`IO::Handle`](https://docs.raku.org/type/IO/Handle) object if it successfully opens the requested file. If it fails, it returns a `Failure`.

This, however, is not what throws the exception — if we actually try to use the `Failure` in an unanticipated way, only **then** the encapsulated exception will be thrown.  In the above example, it's calling the `get` method on the `Failure`.

There are only two ways of preventing the exception inside a `Failure` to be thrown (i.e., anticipating a potential failure):
- Call the [`defined`](https://docs.raku.org/type/Failure#method_defined) method on the Failure
- Call the [`Bool`](https://docs.raku.org/type/Failure#method_Bool) method on the Failure

In either case, these methods will return `False` (even though technically the `Failure` object is instantiated). Apart from that, they will also mark the failure as "handled", meaning that if the `Failure` object is later used in an unanticipated way, it will not throw the exception but simply return `False`.

Calling `defined` or `Bool` on most other instantiated objects will always return `True`. This gives you an easy way to find out if something you expected to return a "real" instantiated object returned something you can really use.

However, it does seem like a lot of work. Fortunately, you do **not** have to explicitly call these methods (unless you really want to). Let's rephrase the above code to more gently handle not being able to open the file:
```
# Raku
my $handle = open "non-existing file";
say "tried to open the file";
if $handle {                 # "if" calls .Bool
    say "we opened the file";
    .say for $handle.lines;  # read/show all lines one by one
}
else {                       # opening the file failed
    say "could not open file";
}
say "but still in business";
# tried to open the file
# could not open file
# but still in business
```

## Throwing exceptions
As in Perl, the simplest way to create an exception in Raku and throw it, is to use the [`die`](https://docs.raku.org/type/Exception#routine_die) function.
```
# Perl
sub alas {
    die "Goodbye cruel world";
    say "this will not be shown";
}
alas;   # Goodbye cruel world at …
```
In Raku, the `die` subroutine provides a shortcut to creating an [`X::AdHoc`](https://docs.raku.org/type/X/AdHoc) exception and throwing it:
```
# Raku
sub alas {
    die "Goodbye cruel world";
    say "this will not be shown";
}
# Goodbye cruel world
#   in sub alas at … #   in …
```
There are some subtle differences with regards to `die` between Perl and Raku, but semantically they are the same: they immediately stop execution.

## Returning with a Failure
Raku has also added the [`fail`](https://docs.raku.org/language/control#fail) function, similar to `die`.

This will also immediately return from the surrounding subroutine/method, with the given Exception but wrapped in a `Failure` object.  If a string is supplied to the `fail` function (rather than an `Exception` object), then an `X::AdHoc` exception with the given string will be created.

Suppose you have a subroutine taking one parameter, a value that is checked for truthiness:
```
# Raku
sub maybe-alas($expected) {
    fail "Not what was expected" unless $expected;
    return 42;
}
my $a = maybe-alas(666);
my $b = maybe-alas("");
say "values gathered";
say $a;                  # ok
say $b;                  # will throw, because it has a Failure
say "still in business"; # will not be shown
# values gathered
# 42
# Not what was expected
#   in sub maybe-alas at …
```
Note that you do not have to provide `try` or `CATCH` block: the `Failure` object will be returned from the subroutine/method in question as if all is normal. Only if the `Failure` object is used in an unanticipated way will the exception embedded in it be thrown.

An alternative way of handling this would be:
```
# Raku
sub maybe-alas($expected) {
    fail "Not what was expected" unless $expected;
    return 42;
}
my $a = maybe-alas(666);
my $b = maybe-alas("");
say "values gathered";
say $a ?? "got $a for a" !! "no valid value returned for a";
say $b ?? "got $b for b" !! "no valid value returned for b";
say "still in business";
# values gathered
# got 42 for a
# no valid value returned for b
# still in business
```
Note that the ternary operator `?? !!` calls the `Bool` method on the condition, so it effectively disarms the `Failure` that was returned by `fail`.

You can think of `fail` as syntactic sugar for immediately returning a `Failure` object:
```
# Raku
fail "Not what was expected";
```
```
# Raku
# semantically the same
return Failure.new("Not what was expected"); 
```

## Creating your own exceptions
Raku makes it very easy to create your own (typed) exception classes. You just need to inherit from the `Exception` class and provide a “message" method. It is customary to make custom classes in the `X::` namespace.

For example:
```
# Raku
class X::Frobnication::Failed is Exception {
    has $.reason;  # public attribute
    method message () {
        "Frobnication failed because of $.reason"
    }
}
```
You can then use that exception in your code in any `die` or `fail` statement:
```
# Raku
die X::Frobnication::Failed.new(
  reason => "too much interference"
);
```
Or wrap it in a `Failure` object:
```
# Raku
fail X::Frobnication::Failed.new(
  reason => "too much interference"
);
```
which you can check for inside a `CATCH` block and introspect if necessary:
```
# Raku
CATCH {
    when X::Frobnicate::Failed {
        if .reason eq 'too much interference' {
            .resume     # too much interference is ok
        }
    }
}                       # all others will re-throw
```
You are completely free in how you set up your exception classes; the only thing the class needs to provide a method called `message` that should return a string. How that string is created is entirely up to you, as long as the method returns a string.

If you prefer working with error codes, you can.  And you don't have to be limited to the named arguments interface if you provide your own `new` method to the `Exception` class:
```
# Raku
my @texts =
  "unknown error",
  "too much interference",
;
my constant TOO-MUCH-INTERFERENCE = 1;
class X::Frobnication::Failed is Exception {
    has Int $.reason;
    method new($reason = 0) { self.bless(:$reason) }
    method message() {
        "Frobnication failed because of @texts[$.reason]."
    }
}
die X::Frobnication::Failed.new(TOO-MUCH-INTERFERENCE);
```
As you can see, this quickly becomes more elaborate, so your mileage may vary.

## Summary
Raku introduces the concept of a `Failure` object, which embeds an `Exception` object. If the `Failure` object is used in an unanticipated way, the embedded exception will be thrown.

You can easily check for a `Failure` with `if`, `?? !!` (which checks for truthiness by calling the `Bool` method) and `with` (which checks for definedness by calling the `defined` method).

You can also create exception classes very easily by inheriting from the `Exception` class and providing a `message` method.
