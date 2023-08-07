#Regular Expressions
Regular expressions have been completely re-imagined from the original Perl regular expressions.  Even to the point that it was decided they were no longer regular in Raku.  Hence the use of the word ["regex"](https://docs.raku.org/language/regexes).

The noticeable differences between Perl and Raku become bigger when the regex becomes more complicated.

## Operator Changes
In Perl, the `=~` (true on match) and `!~` (true if no match):
```
# Perl
say "matched" if "foo" =~ /o/;         # matched
say “did not match" if "foo" !~ /x/;   # did not match
```
operators have been replaced by [`~~`](https://docs.raku.org/language/operators#infix_~~) and `!~~` in Raku:
```
# Raku
say "matched" if "foo" ~~ /o/;         # matched
say "did not match" if "foo" !~~ /x/;  # did not match
```
However, Raku also has some methods that may be better readable if you're just interested in whether there is an occurrence: [`contains`](https://docs.raku.org/type/Str#method_contains), [`starts-with`](https://docs.raku.org/type/Str#method_starts-with), [`ends-with`](https://docs.raku.org/type/Str#method_ends-with) and [`substr-eq`](https://docs.raku.org/type/Str#method_substr-eq).
```
# Raku
say "contains 'o'"     if "foo".contains("o");     # contains 'o'
say "contains letters" if "foo".contains(/ \w+ /); # contains letters
say "starts with 'f'"  if "foo".starts-with("f");  # starts with 'f'
say "ends-with 'o'"    if "foo".ends-with("o");    # ends with 'o'
say "oo from 2nd char" if "foo".substr-eq("oo",1); # oo from 2nd char
```
The same operator changes apply for substitution. Code in Perl would look like:
```
# Perl
my $string = "foo";
$string =~ s/o/x/;
say $string;                  # fxo
```
Just the operator is different in this case in Raku:
```
# Raku
my $string = “foo”;
$string ~~ s/o/x/;
say $string;                  # fxo
```
Newer versionis of Perl also allow the `r` modifier, to just return the substitution rather than attempt to modify the source string:
# Perl
say "foo" =~ s/o/x/r;         # fxo
```
Raku does *not* have that modifier.  But it does have other syntax for achieving the same result, using the [`subst`](https://docs.raku.org/type/Str#method_subst) method:
```
# Raku
say "foo".subst( /o/, “x” );  # fxo
```
Note that the `subst` method does not actually require a regular expression, but can also be used with bare strings only also:
```
# Raku
say "foo".subst( “o”, “x” );  # fxo
```
Which is actually quite a bit more performant as well.

## Whitespace is not significant
In Perl’s regular expressions, whitespace is significant:
```
# Perl
say "matched" if "foo" =~ / o /;       # no output
```
One can add the `x` modifier to make whitespace not significant in Perl:
```
# Perl
say "matched" if "foo" =~ / o /x;      # matched
```
One could consider the `x` modifier to be *always* specified in Raku:
```
# Raku
say "matched" if "foo" ~~ / o /;       # matched
```
If you do want to match a string with possible whitespace, you must quote it in Raku:
```
# Raku
say “matched” if "f o o" ~~ / " o " /; # matched
```
Note that in this case, the whitespace outside of the quoted string, is ignored again.

## Positional Captures
Positional captures in Perl start at number **1**:
```
# Perl
say $1 if "foo" =~ /(.)/;              # f
```
In Raku, they start at **0**, like any other array index:
```
# Raku
say $0 if "foo" ~~ / (.) /;            # ｢f｣
```
This is because `$0` is short-hand for `$/[0]`, in which `$/` is the latest [`Match`](https://docs.raku.org/type/Match) object (the result of the smartmatch with `~~`).

## Named Captures
Perl has support for named captures since version 5.22.  Raku has this from the start.

In Perl, this is achieved with the `(?<name>regex)` syntax, and obtaining the matched string is by accessing the `%+` hash:
```
# Perl
"The colour is blue" =~ /is (?<colour>\w+)/;
say "Found colour '$+{colour}';  # Found colour 'blue'
```
In Raku, the syntax is a little different: `$<name>=(regex)`, and obtaining the matched string uses the same syntax for indicating the name `$<name>`:
```
# Rake
"The colour is blue" =~ /is $<colour>=(\w+)/;
say "Found colour '$<colour>'";  # Found colour 'blue'

## Character Classes
Specification of character classes in Raku is slightly different from Perl, and a little more flexible.  Basically, the square brackets have been repurposed as non-capturing grouping syntax (as that in most cases, will be used more often than character classes).  Also indicating ranges of characters uses the standard [`Range`](https://docs.raku.org/type/Range) syntax in Raku.

Some examples in Perl:
```
# Perl
"foo" =~ /[aeiuo]/;           # match a char of a e i o u
"bar" =~ /[^aeiou]/;          # match a char that is NOT a e i o u
"baz" =~ /[a-z]/;             # match a char between a and z inclusive
"AMS" =~ /[[:upper:]]/;       # match a char that is uppercase
"CMI" =~ /[aeiou[:upper:]]/;  # combined character classes
```
The equivalents in Raku:
```
# Raku
"foo" ~~ /<[aeiuo]>/;         # match a char of a e i o u
"bar" ~~ /<-[aeiou]>/;        # match a char that is NOT a e i o u
"baz" ~~ /<[a..z]>/;          # match a char between a and z inclusive
"AMS" ~~ /<:Upper>/;          # match a char that is uppercase
"CMI" ~~ /<[aeiou]+:upper>/;  # combined character classes
```
And because whitespace is **not** significant inside of the `<[ ]>` either in Raku, you can write them in a much more readable way:
```
# Raku
"foo" ~~ / <[ a e i u o ]>/;
"bar" ~~ / <-[ a e i o u ]>/;
"baz" ~~ / <[ a .. z ]> /;
"AMS" ~~ / <:upper> /;
"CMI" ~~ / <[ a e i o u ] + :upper> /;
```
Note that in Raku you could consider specifying a character class as a grouping of characters, so using `[ ]` inside of the `< >` should make sense mnemonically.

## Under the hood
Under the hood, the regex engines of Perl and Raku couldn't be more different.  In Perl, the regex engine is basically a state machine with extensions that allow for code execution.  In Raku, the regex engine is just executable code.

So when you "run a regex" in Raku, it's really just Raku code that is being executed under the hood.  Not a state machine that is written in another language.

A named [`Regex`](https://docs.raku.org/type/Regex) in Raku is really just a piece of executable code, much like a `method`.  And a [`Grammar`](https://docs.raku.org/language/grammar_tutorial) could be considered a special type of `class` with regexes as methods.

This has implications for [composability](https://en.wikipedia.org/wiki/Composability).  A whole book could probably be written about regexes and grammars of the Raku Programming Language.  And *Moritz Lenz* actually did this in ["Parsing with Regexes and Grammars - A Recursive Descent into Parsing"](https://www.amazon.com/Parsing-Perl-Regexes-Grammars-Recursive/dp/1484232275).

## Summary
This covers the most obvious user visible changes between regexes in Perl and Raku.

There's definitely more to be said about the differences, but it feels the difference in approach to regexes is so large, that doing further comparisons would not be treating either version well.

If you're interested in pursuing Raku regexes further, then these tutorials are recommended reading:
- [Regexes: best practices and gotchas](https://docs.raku.org/language/regexes-best-practices)
- [Grammar tutorial](https://docs.raku.org/language/grammar_tutorial)
