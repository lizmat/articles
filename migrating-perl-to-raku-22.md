# Command-Line Scripts
Historically, Perl has been used a lot for small utility scripts that take a number of parameters, process them and produce a result.  The basic support for command-line argument parsing in the Perl core is really basic, but useful in its own way: especially the magic of `shift` can be very useful for very simple positional command line arguments.

For more complex argument parsing, the Perl ecosystem provides the `Getopt::Long` module.  Raku supplies similar, but extended features by default in its core.  But for those of you wanting to have exactly the same command line argument parsing, there is also a Raku version of the [`Getopt::Long`](https://raku.land/cpan:LEONT/Getopt::Long) module.

This blog post will focus only on how the standard command line parsing features of Raku compare with the standard command line parsing of Perl.

# The Barebones Approach
If you create a command-line script in Perl that takes an integer as a single positional parameter, then the code for that is really very simple:
```
# Perl
my $dial = shift;  # in mainline shifts from command line argument
say “Frobnicator dial is at $dial!”;
```
Assuming this code is stored in a "frobnicator.pl" file, and you call it with the argument "10", then you will see:
```
$ perl frobnicator.pl 10
Frobnicator dial is at 10!
```
The direct equivalent in Raku, is not very different:
```
# Raku
my $dial = shift @*ARGS;  # no magic behaviour of shift
say "Frobnicator dial is at $dial!";
```
However, what if you pass something that is not an integer?  It would just be accepted:
```
$ perl frobnicator.pl foobar
Frobnicator dial is at foobar!
```
Which is probably not what you want.

Also, if you forget to specify an argument, it will just do what you ask it to:
```
$ perl frobnicator.pl
Frobnicator dial is at !
```
Without any indication that you are expected to give a number.

This goes for both the Perl as well as the Raku version of this script.  Although the Raku version will be quite verbose about using a type object in a string:
```
$ raku frobnicator.raku
Use of uninitialized value element of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it
to something meaningful.
  at frobnicator.raku line 1
Frobnicator dial is at !
```
This is because the Raku equivalent of Perl's `use warnings` is **always** activated in Raku.

# A bit more than Bare Bones
Fortunately, in Raku it is very simple to create a more informative script without too much effort.  You can do this by creating a subroutine called [`MAIN`](https://docs.raku.org/language/create-cli#sub_MAIN) (with that exact name) in your code:
```
# Raku
sub MAIN (Int $dial) {
    say “Frobnicator dial is at $dial!”;
}
```
Supposing this code lives in a file called "frobnicator.raku", and you call this without any parameters, it will give you some nice feedback:
```
$ raku frobnicator.raku
Usage:   frobnicator.raku <dial>
```
Which should tell you that it is expecting a single positional value as a parameter.

So how is Raku able to do this?  Well, Raku keeps a lot more information about its internal structures around during execution than Perl does, which allows Raku to introspect itself during execution.

In this particular case, the call to the `MAIN` subroutine could not be made, because its signature (expecting a single positional argument) does **not** match the lack of arguments given on the command line.  It then looks at the signature of the `MAIN` subroutine, and determines that it requires a single positional and creates a feedback message from that information, and prints that on the standard error output.

But Raku doesn’t stop here.  You can add inline documentation to your code that will become visible should they be needed:
```
# Raku
sub MAIN (
  Int $dial  #= The setting of the dial
) {
    say “Frobnicator dial is at $dial!”;
}
```
Note that a comments starting with [`#=`](https://docs.raku.org/language/pod#Declarator_blocks) (a hash symbol followed by a pipe symbol) will attach the given documentation to the identifier preceding it (in this case the `$dial` parameter).

If you now call this script without parameters, the output becomes:
```
$ raku
frobnicator.raku
Usage:   frobnicator.raku <dial>

  <dial>    The setting of the dial (Int)
```
Note that it introspected the inline documentation as well, and combined this with the type information that was available (expecting an `Int`).

Named Parameters
Perl by default, does not really support named parameters on the command line.
Unless you consider this approach named parameters:
```
# Perl
my %named = @ARGV;
say "Frobnicator dial is at $named{dial}!";
```
And then call it with:
```
$ perl frobnicator.pl dial 10
Frobnicator dial is at 10!
```
Raku *does* support named parameters out of the box, mainly because named parameters are also supported in subroutine signatures!

By only a small change in the Raku version of the script, does the dial value become a named variable:
```
# Raku        ↓
sub MAIN (Int :$dial) {  # the colon makes it a named variable
    say "Frobnicator dial is at $dial!";
}
```
With that change, you can call it with:
```
$ raku frobnicator.raku --dial=10
Frobnicator dial is at 10!
```
However, this does not catch the case when the named parameter is not specified:
```
$ raku frobnicator.raku
Use of uninitialized value element of type Any in string context.
Methods .^name, .perl, .gist, or .say can be used to stringify it
to something meaningful.
  in sub MAIN at frobnicator.raku line 1
Frobnicator dial is at !
```
This is because named parameters in Raku are *optional* by default.  They can be made mandatory by adding an exclamation mark to the parameter in the signature:
```
# Raku              ↓
sub MAIN (Int :$dial!) {  # the exclamation mark makes it mandatory
    say “Frobnicator dial is at $dial!”;
}
```
If you now call this script without any parameters, you get better feedback:
```
$ raku frobnicator.raku
  Usage:   frobnicator.raku —dial=<Int>
```
And with inline documentation added:
```
# Raku
sub MAIN (
  Int :$dial!  #= The setting of the dial
) {
    say “Frobnicator dial is at $dial!”;
}
```
Calling this script without parameters, would give you:
```
$ raku frobnicator.raku
Usage:   frobnicator.raku —dial=<Int>

  --dial=<Int>    The setting of the dial
```

## Summary
There is **much** more to be said about the built-in features of creating command-line scripts in the Raku Programming Language.  The ["Creating your own CLI"](https://docs.raku.org/language/create-cli) section in the Raku documentation should be able to help you along with that.
