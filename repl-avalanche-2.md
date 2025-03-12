# Debugging by REPL

> This is part 2 in the [REPL Avalanche](https://dev.to/lizmat/repl-avalanche-45hh) blog series.

The [`REPL`](https://raku.land/zef:lizmat/REPL) distribution also offers a `repl` subroutine apart from a command-line interface (CLI).  A call to this subroutine can be placed in source code during development of a code-base, and then allows for some ad-hoc interactive debugging features.

For the purpose of fun and taste, I will be calling such a (temporary) placement of a call to the `repl` sub a ["**sprinkle**"](https://en.wikipedia.org/wiki/Sprinkles).

> All of the examples in this blog post assume that the `REPL` distribution has been installed, and has been loaded either explicitely with a `use REPL` command in your code, or by setting the `RAKUDO_OPT` environment variable to include `-MREPL`.

## Sprinkling differently
One of the ways you can use the [`REPL`](https://raku.land/zef:lizmat/REPL) distribution is as a debugging tool.  Instead of putting `print` or `dd` statements in your code, you can sprinkle `repl` statements in your code.  And then interactively check out the situation when that `repl` statement is executed.  A very contrived and simplified example:
```
#line 666 foo.raku
my $answer = 42;
if $answer == 42 {
    repl
}
```
would open the REPL with:
```
block at foo.raku line 666
[0] >
```
Note that it does **not** show the standard REPL header, which would look like this:
```
Welcome to Rakudo™ v2025.02.
Implementing the Raku® Programming Language v6.d.
Built on MoarVM version 2025.02.

To exit type '=quit' or '^D'. Type '=help' for help.
```
because that would become *very* annoying for regular users of the `repl` subroutine.

Instead of that, it shows the location of the block in which the `repl` subroutine was called.  This should give you a rough idea of where you are in the code at the moment the `repl` is called.  Which can be handy if you have a number of `repl` calls sprinkled in your code.

If you would like to know the value of the variable `$answer` once in the REPL, then you can enter that variable as you could any other expression:
```
[0] > $answer
42
[1] >
```
Because the `repl` subroutine knows about the runtime context it is being called, it can look up any variable that could be referenced in the location of the code where the `repl` subroutine was called.

Not only can you look up values in variables, you can also change them!  Continiuing the above sprinkle:
```
[1] > $answer = 137
137
[2] > $answer
137
[3] >
```
This allows one to explore "what-if" scenarios, and the like.  Of course, changes are only allowed on things that are changeable, just like they would be in normal execution of code.

## Conditional sprinkles
If you would only like to open a REPL if something unexpected happens, you can of course do that as well.  Again, a contrived example:
```
if 42 -> $answer {
    repl if $answer != 42;
}
```
would only open up a REPL if the answer was wrong.  In this example, that will of course be never.  But I hope you get the picture!

## Identifying sprinkles
Sometimes you are not so much interested in the exact location in the code, but are more interested in a logical step in your code.  To allow easy identification of the sprinkle, you can pass a string in the call to the `repl` subroutine:
```
if 42 -> $answer {
    repl "What did Deep Thought say?";
}
```
would then open the REPL with:
```
What did Deep Thought say?
[0] >
```
If you want to get a little more whimsical: all of the expansions that are possible in the prompt are also possible in this string.  And since we could always use a little fun, why not identify your sprinkle with:
```
if 42 -> $answer {
    repl ":clown-face:";
}
```
which would then open the REPL with:
```
🤡
[0] >
```
Or you could express your feeling about this debugging session with ":confused:" (😕) or ":exploding-head:" (🤯).

> See the [`Text::Emoji`](https://raku.land/zef:lizmat/Text::Emoji#people--body) documentation for some inspiration.

Once in the REPL again you could enter "$answer" to find out the answer.  But that would get pretty boring in a loop pretty quickly!

## Priming sprinkles
If you'd like to know the value of one or more variables or expressions at a specific sprinkle, you can also prime the sprinkle with named arguments.  For instance:
```
if 42 -> $answer {
    repl "Ready!", :$answer, :time<:HHMM:>;
}
```
would then open the REPL with:
```
Ready!
answer: 42
time: 15:10
[0] >
```
which not only would give you the value of the "$answer" variable immediately so you wouldn't have to ask for the answer every time you enter that sprinkle.

And you'd know the time: if the value of a primed expression in the call to `repl` is a string, it will allow for all the expansions that are possible in the prompt as well.  In this example ":HHMM:" expands to the current hour and minute (see [`DateTime::strftime`](https://raku.land/zef:lizmat/DateTime::strftime) for inspiration).

## Fast consumption of sprinkles
Generally, once you're done with a specific sprinkle, you'd like to continue executing your code.  So you'd need to leave the REPL with "=quit" (or its shortening: "=q").  That also can get annoying after a while.

That's why the REPL gets in a special mode when called as a sprinkle.  The "empty" command is then the same as "=quit", so you only need to press ENTER without having entered anything to continue execution.

## Conclusion

By using sprinkles in your code, you can both interactively look up values in your code, as well as immediately see any values / expressions that you would be interested in for a specific sprinkle.

This is the second part of a series of blog posts about the [`REPL`](https://raku.land/zef:lizmat/REPL) distribution.  Future installments will look at the available commands, and the completions logic.

Image courtesy of *Salve J. Nilsen*.

*If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!*
