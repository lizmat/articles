# REPL Avalanche

Sometime in early November, I decided to have a look at re-imagining the REPL (Read, Eval, Print, Loop) that is provided by default by the [Raku Programming Language](https://raku.org).  Little did I know that that work would in the end result in *six* new distributions.  Each providing some useful sub-functionality of the REPL, but also providing useful functionality outside of the REPL context.

> Parallel to this effort, a [Problem Solving Issue](https://github.com/raku/problem-solving/issues/459) was created about the lack of configurability of the standard REPL, and a proof of concept [Pull Request](https://github.com/Raku/problem-solving/pull/460) was made by *Will Coleda*, which proved to be both inspirational.

## The standard REPL

But first, for the uninitiated, a small introduction.  The Raku standard REPL can be invoked by calling `raku` from the command line without any arguments:
```
$ raku
Welcome to Rakudo™ v2025.01.
Implementing the Raku® Programming Language v6.d.
Built on MoarVM version 2025.01.

To exit type 'exit' or '^D'
[0] >
```
And then you can enter Raku code and have it immediately executed.  For example:
```
[0] > my $a = 42
42
[1] > say "a = $a"
a = 42
[1] > say $*0
42
[1] >
```
Note that the number between the square brackets went from `0` to `1` after the first line entered.  But not after the second line entered.  This is because the first line did **not** cause any output.  In that case, the REPL will show the value of the expression and keep it for future reference.

The second line entered *did* cause some output, so the value was **not** saved, and the index was **not** incremented.  Finally, the third line shows how you can refer to the originally saved value, by accessing the dynamic variable `$*0` (with the number corresponding to the index at which the value was saved).

> There is no help available in the standard REPL: it doesn't even have any commands!  The way to exit the REPL is to type "exit".  Which is in fact a Raku function that exits the current process.

## Three months on
Three months on since November 2024, the [`REPL`](https://raku.land/zef:lizmat/REPL) distribution provides a REPL that is ready for production (so to speak, as the use of this tool in production would be limited).

> `zef install REPL` is enough to install this distribution and all of its dependencies.

The `REPL` distribution provides the same features as the standard Raku REPL, but also:
- configurable prompt through command line arguments and environment variables
- REPL specific commands start with "=" to distinguish them from code
- command shortcuts ("=q" being short for "=quit" to exit the REPL)
- help sections for beginners (=introduction, =completions)
- context-sensitive TAB completions
- special purpose TAB completions (\^123 → ¹²³ → ₁₂₃, foo! → FOO → Foo, \heart → 🫀 → 💓 → ...)
- show stack trace if called with "repl" sub (=stack)
- save code entered so far (=write)
- reload code that was saved before (=read)
- edit file inside the repl or code saved (=edit)
- allow creation / switching between contexts (=context)
- installs a command-line script `repl` for direct access
- provides a `REPL` role to facilitate further customization

Over the coming months, a number of additional features will probably be added as well, depending on demand.  But first, let's have a look at all of the features.

## Configuring the prompt

When using the `repl` command line script, one can customize the prompt with the `--the-prompt` and `--symbols` named arguments.  For example:
```
$ repl --the-prompt='[:index:] :HHMM: :symbol: ' --symbols=🦋,🔥
Welcome to Rakudo™ v2025.01.
Implementing the Raku® Programming Language v6.d.
Built on MoarVM version 2025.01.

To exit type '=quit' or '^D'. Type '=help' for help.
[0] 20:51 🦋 if 42 {
[0] 20:51 🔥     say "foo"
[0] 20:51 🔥 }
foo
[0] 20:52 🦋 
```
The `--the-prompt` part of the customization exists of a string that can hold any Unicode codepoints, as well as:
- ANSI color / formatting codes (e.g. `\e[33m`)
- `strftime` escape codes (e.g. `%R`)
- ANSI color placeholder (:yellow:)
- `strftime` placeholder (:HHMM:)
- emoji placeholder specifications (:butterfly:)
- special placeholder (:index:, :symbol:)

> That's quite a lot of possibilities: the [`Prompt::Expand`](https://raku.land/zef:lizmat/Prompt::Expand) distribution provides an overview, as well as the [`Text::Emoji`](https://raku.land/zef:lizmat/Text::Emoji) distribution for the emojis.

The same applies for the `--symbols` argument.  This argument specifies what the `:symbol:` placeholder should be converted to.  This symbol indicates the state of the REPL with regards to code compilation.  Two states are currently recognized:
- ready to start new expression (in this example: 🦋, default ">")
- in the middle of a multi-line expression (in this example: 🔥, default "*").

Note that the different states are separated by a comma in the specification.

If the `--the-prompt` part does not contain a `:symbol:` placeholder, it will be automatically added at the end.  So the above example could also be written as:
```
$ repl --the-prompt='[:index'] %R' --symbols=':butterfly:,:fire:'
```

"That's all nice, but I really don't want to enter all of those arguments every time", you might think, or even say out loud.  Fear not, there are also environment variables that you can set.  The above example could also be written as:
```
$ export RAKUDO_REPL_PROMPT='[:index:] :HHMM:'
$ export RAKUDO_REPL_SYMBOLS=':butterfly:,:fire:'
$ repl
[0] 20:57 🦋
```
If you want, you can put those environment variables in your startup script, so that these settings always apply by default.

> A word of caution: yours truly has spent a lot of time playing with the prompt settings.  It can be a serious time-sink.

## Conclusion

Although it was my original intent that the `REPL` module would be incorporated into the core, it now feels like it has gotten so many features (and depencies) that it has probably become too big to be incorporated in the Raku core.  Which leads to the question whether a subset of the functionality should be incorporated in core, or that maybe the REPL should be removed from core altogether.  And include the `REPL` distribution and its dependencies in all derived packaging, such as Rakudo Star.

This is the first part of a series of blog posts about the `REPL` distribution.  Future installments will look at the available commands, the completions logic, and how the `repl` subroutine can be used in debugging your code.

*If you like what I'm doing, committing to a [small sponsorship](https://github.com/sponsors/lizmat/) would mean a great deal to me!*
