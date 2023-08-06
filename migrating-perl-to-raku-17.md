# Exceptions
This blog post looks at the differences in creating and handling exceptions between Perl and the Raku Programming Language.

## Exception-handling phasers
In Perl, you can use `eval` to catch exceptions in a piece of code:
```
# Perl
eval {
    die "Goodbye cruel world";
};
say $@;           # Goodbye cruel world at …
say "Alive again!";
```
In Raku, this functionality is covered by `try`:
```
# Raku
try {
    die "Goodbye cruel world";
}
say $!;  # Goodbye cruel world␤  in block …
say "Alive again!"
```
In Perl, you can also use the return value of `eval` in an expression:
```
# Perl
my $foo = eval { ... };  # undef if exception was thrown
```
This works the same way in Raku for `try`:
```
# Raku
my $foo = try { 42 / $something };  # Nil if $something is 0
```
and it doesn't even have to be a block:
```
# Raku
my $foo = try 42 / $something;  # Nil if $something is 0
```
In Perl, if you need finer control over what to do when an exception occurs, you can use special signal handlers `$SIG{__DIE__}` and `$SIG{__WARN__}`.

In Raku, these are replaced by two exception-handling phasers, which due to their scoping behaviour must always be specified with curly braces. These exception-handling phasers (in the following table) are applicable only to the surrounding block, and you can have only **one** of each type in a block.
```
  Name      Description
  -----------------------------------------------
  CATCH     Run when an exception is thrown
  CONTROL   Run for any (other) control exception
  -----------------------------------------------
```

## Catching exceptions
The `$SIG{__DIE__}` pseudo-signal handler in Perl is no longer recommended.

There are several competing CPAN modules that provide try/catch mechanisms (such as: `Try::Tiny` and `Syntax::Keyword::Try`). Even though these modules differ completely in implementation, they provide very similar syntax with only very minor semantic differences, so they're a good way to compare Raku and Perl features.

In Perl, you can catch an exception only in conjunction with a `try` block:
```
# Perl
use Try::Tiny;               # or Syntax::Keyword::Try
try {
    die "foo";
}
catch {
    warn "caught error: $_”; # $@ when using Syntax::Keyword::Try
}
```
Raku does **not** require a try block.

The code inside a `CATCH` phaser will be called *whenever* an exception is thrown in the immediately surrounding lexical scope:
```
# Raku
{ # surrounding scope, added for clarity
    CATCH {
        say "aw, died";
        .resume;         # $_, AKA topic, contains the exception
    }
    die "goodbye cruel world";
    say "alive again";
}
# aw, died
# alive again
```
Again, you do **not** need a `try` statement to catch exceptions in Raku.

You can use a `try` block on its own, if you want, but it's just a convenient way to disregard any exceptions thrown inside that block.

Also, note that `$_` will be set to the [`Exception`](https://docs.raku.org/type/Exception) object inside the `CATCH` block.

In this example, execution will continue with the statement after the one that caused the exception to be thrown. This is achieved by calling the [`resume`](https://docs.raku.org/type/Exception#method_resume) method on the `Exception` object.

If the exception is not resumed, it will be thrown again and possibly caught by an outer `CATCH` block (if there is one). And if there are no outer `CATCH` blocks, the exception will result in program termination.

The [`when`](https://docs.raku.org/language/control#when) statement makes it easy to check for a specific exception:
```
# Raku
{
    CATCH {
        when X::NYI {  # Not Yet Implemented exception thrown
            say "aw, too early in history";
            .resume;
        }
        default {
            say "WAT?";
            .rethrow;  # throw the exception in $_ again
        }
    }
    X::NYI.new(feature => “Frobnicator").throw; # resumed
    now / 0;                                    # rethrown
    say "back to the future”;                   # never gets here
}
# aw, too early in history
# WAT?
# Attempt to divide 1234.5678 by zero using /
```
In this example, only exceptions of the `X::NYI` type will resume; all the others will be thrown to any outer `CATCH` block and will probably result in program termination. And we'll never go back to the future.

## Catching warnings
If you do not want any warnings to emanate when a piece of code executes, you can use the `no warnings` pragma in Perl:
```
# Perl
use warnings;     # need to enable warnings explicitly
{
    no warnings;
    my $foo;
    say $foo;     # no visible warning
}
my $bar;
print $bar;  # Use of uninitialized value $bar in print...
```
In Raku, you can use a `quietly` block:
```
# Raku
                # warnings are enabled by default 
quietly {
    my $foo;
    say $foo;   # no visible warning
}
my $bar;
print $bar;
# Use of uninitialized value of type Any in string context
```
The `quietly` block will catch any warnings that emanate from that block and disregard them.

If you want finer control in Perl on which warnings you want to see, you can select the warning categories you want enabled or disabled with `use warnings` or `no warnings`, respectively:

For example:
```
# Perl
use warnings;
{
    no warnings 'uninitialized';
    my $bar;
    print $bar;    # no visible warning
}
```
If you want to have finer control in Raku, you will need a `CONTROL` phaser.

## CONTROL Phaser
The `CONTROL` phaser is very much like the `CATCH` phaser, but it handles a special type of exception called the "control exception." A control exception is thrown whenever a warning is generated in Raku, which you can catch with the `CONTROL` phaser.

This example will not show warnings for using uninitialised values in expressions:
```
# Raku
{
    CONTROL {
        when CX::Warn {  # Control eXception type for Warnings
            note .message
              unless .message.starts-with('Use of uninitialized');
        }
    }
    my $bar;
    print $bar;          # no visible warning
}
```
There are currently no warning categories defined in Raku, but they are being discussed for future development. In the meantime, you will have to check for the actual message of the control exception `CX::Warn` type, as shown above.

The control exception mechanism is used for quite a lot of other functionality in addition to warnings. The following statements (in alphabetical order) also create control exceptions:

- emit
- fail
- last
- next
- proceed
- redo
- return
- return-rw
- succeed
- take

Control exceptions generated by these statements will also show up in any `CONTROL` phaser. Luckily, if you don't do anything with the given control exception, it will be re-thrown when the local `CONTROL` phaser is finished and ensure its intended action is performed.

## Summary
Catching exceptions and warnings are handled by phasers in Raku, not by `eval` or signal handlers, as in Perl. `Exception`s are first-class objects in Raku.
