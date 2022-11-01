# It's time to rak! (Part 3)

This blog post will discuss the types of patterns you can specify with [`rak`](https://raku.land/zef:lizmat/App::Rak).

## The --type argument

To start right off the bat: the `--type` argument indicates how a given pattern (specified as a string on the command line) should be interpreted.  Having to specify `--type=foo` all of the time, can be bothersome.  So there are a number of shortcuts that you can use to make your life easier.  These will be discussed at the end.

The results in all these examples, are based on the existence of a file called "twenty" in the current directory, which contains the words "one", "two", "three" ... "twenty", each on a separate line.  Because the contents of the line matches the line number, it hopefully makes understanding the example output in this post easier.

Note that lines will be matched **without** the end-of-line by default.

### --type=contains

Match all lines that contain the literal pattern **anywhere**.
```
# Look for "ve" anywhere on any line in file "twenty"
$ rak --type=contains ve twenty
twenty
5:fi𝐯𝐞
7:se𝐯𝐞n
11:ele𝐯𝐞n
12:twel𝐯𝐞
17:se𝐯𝐞nteen
```
Note that first the filename is shown, and then the actual matches with the pattern (𝐯𝐞) highlighted in the result.  This is *on* by default, if `rak` is being run by a human (aka, there's actually someone at the keyboard).  Or more technically, if STDIN is connected to a TTY.

Also note that the matched lines are prefixed with their line number (which happens to be the same as the textual version of the number).

> Unfortunately, markdown (in which this blog post is written) does not allow bolding of characters inside code blocks.  So I've had to resort to using the equivalent "𝐛𝐨𝐥𝐝" codepoints (versus "normal" and "**bold**"), which may render not quite as I intended.  Hopefully they will get the message across anyway.

### --type=words

Match any line that contains the literal pattern as a **word**.  A word here is defined as a string consisting of alphanumeric characters, with non-alphanumeric characters (or the absence of a character) on both ends.
```
# Look for "six" as a word on any line in file "twenty"
$ rak --type=words six twenty
twenty
6:𝐬𝐢𝐱
```

In this case "six" was acceptable, because there are no characters before or after on the line.  So in that sense, it is a bad example of using "words" as a type of search.  Note that "sixteen" was *not* matched, because there was no word-boundary at "x".

### --type=starts-with

Match any line that **starts** with the literal pattern.
```
# Look for "seven" at the start of all lines in file "twenty"
$ rak --type=starts-with seven twenty
twenty
7:𝐬𝐞𝐯𝐞𝐧
17:𝐬𝐞𝐯𝐞𝐧teen
```

Note that in this case the line with "seventeen" *was* matched, because it starts with "seven".

### --type=ends-with

Match any line that **ends** with the literal pattern.
```
# Look for "ve" at the end of all lines in file "twenty"
$ rak --type=ends-with ve twenty
twenty
5:fi𝐯𝐞
12:twel𝐯𝐞
```

Note that in this case the lines with "seven", "eleven" and "seventeen" were *not* matched, because they didn't end with "ve", even though they had the literal string in them.

### --type=equal

Match any line that matches in its **entirety**  with the literal pattern.
```
# Look for "eight" as the whole line in file "twenty"
$ rak --type=equal eight twenty
twenty
8:𝐞𝐢𝐠𝐡𝐭
```

Note that in this case the line with "eighteen" was *not* matched, because it had additional characters after "eight".

### --type=regex

Match any line that matches the literal pattern as a **regex**.
```
# Look for strings starting with "e" and ending with "t" and any
# number of characters between them (.*), in file twenty
$ rak --type=regex 'e.*t' twenty
twenty
8:𝐞𝐢𝐠𝐡𝐭
17:s𝐞𝐯𝐞𝐧𝐭een
18:𝐞𝐢𝐠𝐡𝐭een
19:nin𝐞𝐭een
20:tw𝐞𝐧𝐭y
```

Note that because of shell issues, you will most likely need to quote this string, because `.` and `*` have special meanings in most shells.

We will get back to what you can do with Raku regexes at the end of this blog post.

### Ignoring case and accents

All of the above values of the `--type` arguments adhere to the `--ignorecase`, `--ignoremark` and `--smartcase` flags.

#### --ignorecase

If this flag is specified, then any matching will be done without regards to case.  In other words "E" will match "e", and vice-versa.
```
# Look for strings containing y or Y
$ rak --type=contains --ignorecase Y twenty
twenty
20:twent𝐲
```

#### --ignoremark

If this flag is specified, then any matching will be done by comparing base characters, and ignore additional marks such as combining accents.  In other words "é" will match "e", and vice-versa.
```
# Look for strings matching "u" having any additional marks
$ rak --type=contains --ignoremark ú twenty
twenty
4:fo𝐮r
```

#### --smartcase

If this flag is specified, then the pattern is checked for uppercase characters.  If none are found, then it acts as if `--ignorecase` has been specified.  Otherwise it does not perform any action.

### --type=code

Convert the pattern into Raku code and run the code for each line.  If it returns `True`, it will consider the line matched.  If it returns `False`, or [`Empty`](https://docs.raku.org/syntax/Empty), or [`Nil`](https://docs.raku.org/type/Nil) it will consider the line did **not** match.

If it returns a [`Slip`](https://docs.raku.org/type/Slip), it will produce the elements of the slip as separate items, and consider the line matched.

If it returns anything else, it will produce that value (and consider the line matched).

```
# Look for lines containing "ve" and show them in uppercase
$ rak '.uc if .contains("ve")' twenty --type=code
twenty
5:FIVE
7:SEVEN
11:ELEVEN
12:TWELVE
17:SEVENTEEN
```
Note that no highlighting can occur when the pattern is code, as it cannot indicate where there was a match (if any).

```
# Look for lines that contain "y" and produce all of its letters separately
$ rak '.comb.Slip if .contains("y")' twenty --type=code
twenty
20:t
20:w
20:e
20:n
20:t
20:y
```
Note that returning a `Slip`, will produce multiple items for the same line.

```
# Convert all occurrences of "teen" into "10"
$ rak '.subst("teen",10)' twenty --type=code
twenty
1:one
2:two
3:three
4:four
5:five
6:six
7:seven
8:eight
9:nine
10:ten
11:eleven
12:twelve
13:thir10
14:four10
15:fif10
16:six10
17:seven10
18:eigh10
19:nine10
20:twenty
```

The [`.subst` for "substitute"](https://docs.raku.org/routine/subst) method is a very handy tool to convert a target into something else.  The target can also be a [`regex`](https://docs.raku.org/language/regexes).

Code patterns are typically used with options such as `--unique`, `--

1. True       produce item, unless --invert-match was specified
2. False      do **NOT** produce item, unless --invert-match was specified
3. Empty      do **NOT** produce item, unless --invert-match was specified
4. Nil        do **NOT** produce item, unless --invert-match was specified
5. Slip       produce all items in the Slip, always
6. product    produce that, always
Match any line that matches the literal pattern as a **regex**.

### --type=auto

## Shortcuts

These shortcuts are only available if `--type=auto` is explicitely specified, or **no** `--type` argument has been specified.

### string

match if "string" occurs anywhere in a line

### §string

match if "string" occurs as a **word** in a line

### ^string

match if "string" occurs at the **start** of a line

### string$

match if "string" occurs at the **end** of a line

### ^string$

match if "string" is equal to a line

### / regex /

match if regex matches a line

### { code }

match if code called with line returns True

### \*code
match if whatever-curried code called with line returns True

Basically, you can recognize three types of patterns that you can specify: literal string, regex or code.

## Regex

Before we go on with how you can use regexes in `rak`, a small detour.

> Everybody knows regular expressions, right?  Well, I guess that's true to some extent.  But regular expressions haven't been regular for a long time already.  That's why it was decided to just call them ["regexes"](https://docs.raku.org/language/regexes) in the Raku Programming Language.

> Regexes in Raku look like your average regular expression: some recipe to tell a state machine to look for matches.  The biggest difference with other regular expression implementations, is that Raku regexes are **code** under the hood that you specify with Raku's regex syntax.  You can think of a regex as a piece of code that you can call inside another regex, like a subroutine or a method.  This has all sorts of implications, but that will be for another series of blog posts.


Regexes are recognized as a pattern if they start *and* end with a slash (`/`).  Some examples:
```
# Look for the string "foo"
$ rak '/ foo /'

# Look for the string "bar" at the start of a line
$ rak '/^ bar /'

# Look for the string "baz" at the end of a line
$ rak '/ baz $/'

# Look for lines consisting of "Example:" or "Examples:"
$ rak '/^ Example s? ":" $/'

# Look for lines with either "foo" or "bar"
$ rak '/ foo | bar /'

# Look for lines with words that start with "spec"
$ rak '/ << spec \w* /'

Whitespace between the slashes will be ignored.  Any non-alphanumeric character needs to be quoted.

Some tips for a quick understanding of Raku regexes

## Code

## Literal string

## Performance tweak

If you're searching large files, e.g. log files, and you are sure that the files are encoded using Latin-1, you can specify C<--encoding=latin1> as a parameter.  This typically makes searching about 2x as fast, as Raku then doesn't need to interpret the contents into Normalization Form Grapheme (NFG).  This also makes comparisons to similar programs more fair, as none of the other programs support NFG.

## Conclusion

This concludes part 3 of a series of blog posts about `rak`.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
