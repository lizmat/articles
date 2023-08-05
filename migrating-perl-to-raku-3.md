# Syntactic Changes between Perl and Raku (Part 2 of 3)

## Ternaries
Ternaries in Raku are formed with `??` and `!!` rather than `?` and `:` in Perl.
```
say $condition ?  "True" :  "False";  # Perl
say $condition ?? "True" !! "False";  # Raku
```
It’s really to free up the colon `:` for a lot of other uses in Raku.  But more about that later.

## Matching
The most obvious change is that Perl uses `=~` to match a string with a regular expression:
```
# Perl
say "matched" if "foo" =~ /o/;  # matched
```
In Raku, the more generic `~~` (aka `smartmatch`) infix operator is used:
```
# Raku
say "matched" if "foo" ~~ /o/;  # matched
```
If your regular expression in Perl is simple enough, there’s a good chance it will just work in Raku (as seen above).  Whenever you use positional captures, there’s a bit of a gotcha involved.

In Perl, positional captures start with `$1`:
```
# Perl
"foo" =~ /(\w)/;say $1;   # f
```
In Raku, positional captures start at `$0`.  This makes sense if you realize that `$0` is syntactic sugar for `$/[0]`, in which the `Match` variable `$/` is used as an array.  And indices of arrays start at `0`.
```
# Raku
"foo" ~~ /(\w)/;say $0;   # ｢f｣
```
Also note that in Raku, any capture is a full `Match` object, that stringifies with the `｢｣` special quotation marks.

## Array and Hash Elements
In Raku, the sigil of an array or a hash does **not** change, regardless of how it is used.  In Perl, the sigil of an array or hash **changes** depending on the context.
```
# Perl
my @array = (1,2,3,4,5);
say $array[0];      # 1, sigil changed
say @array[1,2,3];  # 234
```
Note how in Perl the sigil changed from `@` to `$` to get a single element out of the array.  In Raku, there is "sigil invariance", the sigil never changes:
```
# Raku
my @array = 1,2,3,4,5;
say @array[0];      # 1, no sigil change
say @array[1,2,3];  # 234
```
The same for hashes:
```
# Perl
my %hash = (a => 42, b => 666);
say $hash{a};    # 42, sigil changed
say @hash{a,b};  # 42666
```
Note that in the case of hashes, a hash in Perl can have 3 different sigils.  This has been a large part of newbie confusion in Perl.  Which is why Raku does it differently, by **not** changing the sigil:
```
# Raku
my %hash = a => 42, b => 666;
say %hash<a>;    # 42, sigil does not change
say %hash<a b>;  # (42 666)
```
Also note that when specifying hash elements in Raku, the closest thing to using bare words is the [`< >`](https://docs.raku.org/language/operators#term_%3C_%3E) syntax.  This is closest to `qw( )` in Perl.

## Curly Close at end of line
In Perl, some structures that take a block (such as `if` and `while`)  do not need a semi-colon after them.  While others, such as `eval`, do need a semi-colon to prevent a syntax error:
```
# Perl
if ($condition) {
    …   # do something
}       # no semi-colon needed
eval {
    …   # some code that might die
};      # requires a semi-colon
```
In Raku, the rule with regards to a closing curly brace is that if it is the **last** non-whitespace character in source code on a line, then a semi-colon will **always** be assumed to follow it.

This means that all code constructs that take a block, can be written like they are an `if`:
```
# Raku
try {
    …   # some code that might die
}       # no semi-colon needed
```
This also implies that if you create a subroutine that takes a block to execute, you do not have to specify a semi-colon either:
```
# Raku
sub call-times ($times, &code) {
    code() for 1 .. $times;
}
call-times 5, {
    say “hello world”;
}       # no semi-colon needed either
```
This may however catch you off-guard in some situations.  Suppose you have an intricate expression involving a key in a hash that you want to split over multiple lines:
```
# Raku
my $result = %hash{$key}  # assumes semi-colon, so end of line
 + 42;                    # a bare expression, will be flagged
```
The only way to work around this is to either **not** do that:
```
# Raku
my $result =              # prevent bare } at end of line
  %hash{$key} + 42;
```
Or you can use the [`unspace`](https://docs.raku.org/language/syntax#Unspace) feature to prevent the bare curly close at the end of the line.
```
# Raku
my $result = %hash{$key} \  # use unspace to prevent bare }
  + 42;
```
Which method you use, is really a matter of taste.  Or possibly some code standard that you enforce on yourself or your co-workers.

## Summary

In this blog post the different syntax for ternaries, the different indexing on matches, the sigil invariance on arrays and hashes, and the standard behaviour of curly close brackets at the end of a line were covered.
