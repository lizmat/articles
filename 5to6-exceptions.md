Failure is an option in Perl 6
==============================

This is the eighth article in
[a series of articles](https://opensource.com/users/lizmat) about migrating
code from Perl 5 to Perl 6.  In this article we'll be looking at the
differences in creating and handling of exceptions between Perl 5 and Perl 6.

The first part of this blog post will be about how to work with exceptions
in Perl 6.  The second part will be about how you can create your own
exceptions, and how failure *is* an option in Perl 6.

Exception handling phasers
--------------------------
In Perl 5 you can use [`eval`](https://perldoc.perl.org/functions/eval.html)
to catch exceptions in a piece of code.  In Perl 6 this functionality is
covered by
[try](https://docs.perl6.org/language/exceptions#index-entry-try_blocks):

    # Perl 5
    eval {
        die "Goodbye cruel world";
    };
    say $@;           # Goodbye cruel world at ...
    say "Alive again!"

    # Perl 6
    try {
        die "Goodbye cruel world";
    }
    say $!;           # Goodbye cruel world␤  in block ...
    say "Alive again!"

In Perl 5 you can also use the return value of `eval` in an expression:

    # Perl 5
    my $foo = eval { ... };  # undef if exception was thrown

That works the same way in Perl 6 for `try`:

    # Perl 6
    my $foo = try { 42 / $something };   # Nil if $something is 0

Which even doesn't have to be a block:

    # Perl 6
    my $foo = try 42 / $something;       # Nil if $something is 0

If you however need finer control over what to do when an exception occurs,
you can use special [signal handlers](https://perldoc.pl/variables/%25SIG)
`$SIG{__DIE__}` and `$SIG{__WARN__}` in Perl 5.

In Perl 6, these are replaced by two exception handling phasers, which due
to their scoping behaviour, *must* always be specified using curly braces.
These exception handling phasers are only applicable to the surrounding block.
And you can only have **one** of each type in a block.  They are:

| Name | Description |
|:----------|:-----------|
| CATCH   | run when an exception is thrown |
| CONTROL | run for any (other) control exception |

Catching exceptions
-------------------
The use of `$SIG{__DIE__}` pseudo-signal handler in Perl 5 is not recommended
anymore.  Several competing CPAN modules provide some kind of `try / catch`
mechanism (such as: [Try::Tiny](https://metacpan.org/pod/Try::Tiny) and
[Syntax::Keyword::Try](https://metacpan.org/pod/Syntax::Keyword::Try)).
However, there isn't really a single Perl 5 target to compare Perl 6 features
against.  So in this case only Perl 6 code will be shown.

The code inside a [`CATCH`](https://docs.perl6.org/language/phasers#CATCH)
phaser will be called whenever an exception is thrown in the immediately
surrounding lexical scope.

    # Perl 6
    {
        CATCH {
            say "aw, died";
            .resume;         # $_, AKA the topic, contains the exception
        }
        die "goodbye cruel world";
        say "alive again";
    }
    # aw, died
    # alive again

Please note that you do **not** need a `try` statement to be able to catch
exceptions.  A `try` block is just a convenient way to disregard any
exceptions without having to worry about anything.

Also note that `$_` will be set to the `Exception` object inside the `CATCH`
block.  In this example, the `Exception` is simply resumed by calling the
`resume` method on the `Exception` object.  Execution will then continue
with the statement after the statement that caused the exception.  If the
exception is not resumed, it will be thrown again, to be possibly caught
by an outer `CATCH` block (if any).

Checking for specific exception is made easy with the
[`when`](https://docs.perl6.org/language/control#default_and_when) statement:

    # Perl 6
    {
        CATCH {
            when X::NYI {       # Not Yet Implemented exception thrown
                say "aw, too early in history";
                .resume;
            }
            default {
                say "WAT?";
                .rethrow;       # throw the exception again
            }
        }
        X::NYI.new(feature => "Frobnicator").throw;  # caught, resumed
        now / 0;                                     # caught, rethrown
        say "back to the future";
    }
    # aw, too early in history
    # WAT?
    # Attempt to divide 1234.5678 by zero using /

In this example only `X::NYI` exceptions will be resumed, all other ones will
be thrown to any outer `CATCH` block and as such will probably result in
program termination.  And we'll never go back to the future.

Catching warnings
-----------------
If you do not want any warnings to emanate from the execution of a piece of
code, you can use the `no warnings` pragma in Perl 5:

    # Perl 5
    use warnings;     # need to enable warnings explicitely
    {
        no warnings;
        my $foo;
        say $foo;     # no visible warning
    }
    my $bar;
    print $bar;
    # Use of uninitialized value $bar in print...

In Perl 6 you can use a `quietly` block:

    # Perl 6
                      # warnings are enabled by default
    quietly {
        my $foo;
        say $foo;     # no visible warning
    }
    my $bar;
    print $bar;
    # Use of uninitialized value of type Any in string context...

The `quietly` block will catch **any** warnings that emanate from that block
and will just disregard them.

If you want finer control on which warnings you want to be seen, you can
select which
[warning categories](https://perldoc.pl/warnings#Category-Hierarchy) you
want enabled / disabled with `use warnings` / `no warnings` in Perl 5.
If you want to do that in Perl 6 however, you will need a `CONTROL` phaser.

CONTROL
-------
The `CONTROL` phaser is very much like the `CATCH` phaser, but it handles
a special type of exception, the so-called "control exception".  Whenever
a warning is generated in Perl 6, a control exception is thrown.  Which
you can catch with the `CONTROL` phaser.  Here's an example that will not
show warnings for using uninitialized values in expressions.

    # Perl 6
    CONTROL {
        when CX::Warn {    # Control eXception type associated with Warnings
            note .message
              unless .message.starts-with('Use of uninitialized value');
        }
    }

There are currently no warning categories defined in Perl 6.  So you will
have to check for the actual `message` of the control exception of
type `CX::Warn`, as shown above.

The control exception mechanism is also used for quite a lot of other
functionality apart from warnings.  The following statements also create
control exceptions (in alphabetical order): 
[emit](https://docs.perl6.org/language/control#supply/emit),
[fail](https://docs.perl6.org/language/control#fail),
[last](https://docs.perl6.org/language/control#last),
[next](https://docs.perl6.org/language/control#next),
[proceed](https://docs.perl6.org/language/control#proceed),
[redo](https://docs.perl6.org/language/control#redo),
[return](https://docs.perl6.org/language/control#return),
[return-rw](https://docs.perl6.org/language/control#return-rw),
[succeed](https://docs.perl6.org/language/control#succeed),
[take](https://docs.perl6.org/language/control#gather/take).  So control
exceptions generated by these statements will also show up in any `CONTROL`
phaser.  Luckily, if you do not do anything with the given control exception,
it will be rethrown when the `CONTROL` phaser is done.  And that will make
sure its intended action will be performed.

Failure is an option
====================
In Perl 5, you either need to prepare for a possible exception (by using
`eval`, or some version of `try` when using a CPAN module).  In Perl 6 you
can do the same with `try`, as we've seen earlier here.

But Perl 6 also has another option:
[Failure](https://docs.perl6.org/type/Failure).  The `Failure` class is a
special class for wrapping an
[Exception](https://docs.perl6.org/type/Exception).  Whenever a `Failure`
object is used in an unanticipated way, it will throw the `Exception` that
it is wrapping.  A simple example:

    # Perl 6
    my $handle = open "non-existing file";
    say "we tried to open the file";
    say $handle.get;  # unanticipated use of $handle, throws exception
    say "this will never be shown";
    # we tried to open the handle
    # Failed to open file non-existing file: No such file or directory

The [open](https://docs.perl6.org/routine/open#(IO)_sub_open) function in
Perl 6 returns a `Failure` if the given file could not be opened.  This in
itself does **not** throw the exception.  However, if we actually try to
use the value returned by `open`, *then* the exception will be thrown.

If opening the file was successful, then an
[IO::Handle](https://docs.perl6.org/type/IO::Handle) will be returned.

There are only **two** ways of preventing the `Exception` inside a `Failure`
to be thrown (AKA anticipating a potential failure):

- call the `.defined` method on the Failure
- call the `.Bool` method on the Failure

In either case, these methods will return `False` (even though technically
the `Failure` object *is* instantiated).  Apart from that, they will **also**
mark the `Failure` as "handled", meaning that if the `Failure` is used in an
unanticipated way afterwards, it will **not** throw the `Exception`.

Calling `.defined` or `.Bool` on most other instantiated objects, will always
return `True`.  So this gives you an easy way to find out if something that
you expected to return a "real" instantiated object, really did return
something that you could actually use.

Having said this, it does seem like a lot of work.  Fortunately you don't
have to explicitely call these methods (unless you really want to).  Let's
rephrase the above code to more gently handle not being able to open the file:

    # Perl 6
    my $handle = open "non-existing file";
    say "tried to open the file";
    if $handle {                    # "if" calls .Bool, True on an IO::Handle
        say "we opened the file";
        .say for $handle.lines;     # read/show all lines one by one
    }
    else {                          # opening the file failed
        say "could not open file";
    }
    say "but still in business";
    # tried to open the file
    # could not open file
    # but still in business

Throwing exceptions
-------------------
As in Perl 5, the simplest way to create an exception and throw it, is to use
the [die](https://docs.perl6.org/routine/die) function.  In Perl 6, this is
a shortcut to creating an [X::AdHoc](https://docs.perl6.org/type/X::AdHoc)
exception and throwing it.

    # Perl 5
    sub alas {
        die "Goodbye cruel world";
        say "this will not be shown";
    }
    alas;
    # Goodbye cruel world at ...

    # Perl 6
    sub alas {
        die "Goodbye cruel world";
        say "this will not be shown";
    }
    # Goodbye cruel world
    #   in sub alas at ...
    #   in ...

There are some subtle differences between `die` between Perl 5 and Perl 6,
but semantically, they are the same: they immediately stop execution.

Returning with a failure
------------------------
Perl 6 has added the [fail](https://docs.perl6.org/syntax/%20fail) function.
This will immediately return from the surrounding subroutine / method with
the given `Exception`: if one only supplies a string, then an `X::AdHoc`
exception will be created for you.

Suppose we have a subroutine taking one parameter: a value that is checked
for truthiness:

    # Perl 6
    sub maybe-alas($expected) {
        fail "Not what was expected" unless $expected;
        return 42;
    }
    my $a = maybe-alas(666);
    my $b = maybe-alas("");
    say "values gathered";
    say $a;                   # ok
    say $b;                   # will throw, because it has a Failure
    say "still in business";  # will not be shown
    # values gathered
    # 42
    # Not what was expected
    #   in sub maybe-alas at ...

Note that you do not have to provide any `try` or `CATCH`: the `Failure` will
simply be returned from the subroutine / method in question as if all is
normal.  Only if the `Failure` is used in an unanticipated way, will it
actually throw the `Exception` that is embedded in it.  An alternative way
of handling this would have been:

    # Perl 6
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

Note that the ternary operator `?? !!` calls `.Bool` on the condition, so
effectively disarms the `Failure` that was returned by `fail`.

One can think of `fail` as syntactic sugar for returning a `Failure` object:

    # Perl 6
    fail "Not what was expected";
    return Failure.new("Not what was expected");  # semantically the same

Creating your own Exceptions
----------------------------
Perl 6 makes it very easy to create your own (typed) `Exception` classes.
You only need to inherit from the `Exception` class, and provide a `message`
method.  It is customary to make custom classes in the `X::` namespace.
For example:

    # Perl 6
    class X::Frobnication::Failed is Exception {
        has $.reason;  # public attribute
        method message() {
            "Frobnication failed because of $.reason"
        }
    }

You can then use that exception in your code in any `die` or `fail` statement:

    # Perl 6
    die X::Frobnication::Failed.new( reason => "too much interference" );

    # Perl 6
    fail X::Frobnication::Failed.new( reason => "too much interference" );

Which you can then check for inside a `CATCH` block, and introspect if
necessary:

    # Perl 6
    CATCH {
        when X::Frobnicate::Failed {
            if .reason eq 'too much interference' {
                .resume     # too much interference is ok
            }
        }
    }                       # all others will re-throw

You are completely free in how you set up your `Exception` classes: the only
thing the class needs to provide is a `message` method that should return a
string.  How that string is created, is entirely up to you.  If you prefer
working with error codes, you can.  As long as the `method` message returns
a string.

    # Perl 6
    my @texts =
      "unknown error",
      "too much interference",
    ;
    my constant TOO_MUCH_INTERFERENCE = 1;
    class X::Frobnication::Failed is Exception {
        has Int $.reason = 0;
        method message() {
            "Frobnication failed because of @texts[$.reason]"
        }
    }

As you can see, this quickly becomes much more elaborate.  So your mileage
may vary.

Summary
=======
Catching exceptions and warnings are also handled by phasers in Perl 6,
rather than by `eval` or signal handlers as in Perl 5.  `Exception`s are
first class objects in Perl 6.

Perl 6 also introduces the concept of a `Failure` object, which embeds an
`Exception` object.  If the `Failure` object is used in an unanticipated way,
then the embedded `Exception` will be thrown.

One can easily check for a `Failure` with `if`, `?? !!` (which
check for truthiness by calling the `.Bool` method) and `with` (which checks
for definedness by calling the `.defined` method).

One can very easily create ones own `Exception` classes by inheriting from
the `Exception` class and providing a `message` method.
