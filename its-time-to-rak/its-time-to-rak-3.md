# It's time to rak! (Part 3)

This blog post will discuss the types of patterns you can specify with [`rak`](https://raku.land/zef:lizmat/App::Rak) as part 3 of the [It's time to rak!](https://dev.to/lizmat/its-time-to-rak-part-1-30ji) blog series.

## The --type argument

To start right off the bat: the `--type` argument indicates how a given pattern (specified as a string on the command line) should be interpreted.  Having to specify `--type=foo` all of the time, can be bothersome.  So there are a number of shortcuts that you can use to make your life easier.  These will be discussed at the end.

The results in all these examples are based on the existence of a file called "twenty" in the current directory, which contains the words "one", "two", "three" ... "twenty", each on a separate line.  Because the contents of the line matches the line number, it hopefully makes understanding the example output in this post easier.

Note that lines will be matched **without** the end-of-line by default.

### --type=contains

The "contains" value on `--type` indicates that a line will match if they contain the given literal pattern **anywhere** on the line.
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

The "words" value on `--type` indicates that a line will match if it contains the literal pattern as a **word**.  A word here is defined as a string consisting of alphanumeric characters, with non-alphanumeric characters (or the absence of a character) on both ends.
```
# Look for "six" as a word on any line in file "twenty"
$ rak --type=words six twenty
twenty
6:𝐬𝐢𝐱
```

In this case "six" was acceptable, because there are no characters before or after on the line.  So in that sense, it is a bad example of using "words" as a type of search.  Note that "sixteen" was *not* matched, because there was no word-boundary at "x".

### --type=starts-with

The "starts-with" value on `--type` indicates that a line will match if it **starts** with the literal pattern.
```
# Look for "seven" at the start of all lines in file "twenty"
$ rak --type=starts-with seven twenty
twenty
7:𝐬𝐞𝐯𝐞𝐧
17:𝐬𝐞𝐯𝐞𝐧teen
```

Note that in this case the line with "seventeen" *was* matched, because it starts with "seven".

### --type=ends-with

The "ends-with" value on `--type` indicates that a line will match if it **ends** with the literal pattern.
```
# Look for "ve" at the end of all lines in file "twenty"
$ rak --type=ends-with ve twenty
twenty
5:fi𝐯𝐞
12:twel𝐯𝐞
```

Note that in this case the lines with "seven", "eleven" and "seventeen" were *not* matched, because they didn't end with "ve", even though they had the literal string in them.

### --type=equal

The "equal" value on `--type` indicates that a line will be accepted if it matches in its **entirety**  with the literal pattern.
```
# Look for "eight" as the whole line in file "twenty"
$ rak --type=equal eight twenty
twenty
8:𝐞𝐢𝐠𝐡𝐭
```

Note that in this case the line with "eighteen" was *not* matched, because it had additional characters after "eight".

### --type=regex

The "regex" value on `--type` indicates that a line will be accepted if it matches the literal pattern as a **regex**.
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

The "code" value on `--type` instructs `rak` to convert the pattern into Raku code and run the code for each line.  If it returns `True`, it will consider the line matched.  If it returns `False`, or [`Empty`](https://docs.raku.org/syntax/Empty), or [`Nil`](https://docs.raku.org/type/Nil) it will consider the line did **not** match.

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

The [`.subst` for "substitute"](https://docs.raku.org/routine/subst) method is a very handy tool to convert a target into something else, because it will return the original string if no match was found.  The target in `.subst` can also be a [`regex`](https://docs.raku.org/language/regexes).

Code patterns are typically used with options such as `--unique` and `--modify-files` and similar.
```
# Produce the frequencies of the letters in file "twenty"
$ rak 'slip .comb' twenty --type=code --frequencies
33:e
17:n
15:t
9:i
5:f
5:v
4:h
4:o
4:r
4:s
3:w
2:g
2:l
2:u
2:x
1:y
```

### --type=auto

This is the default type if there is no `--type` specified.  It indicates that the given pattern(s) should be inspected for shortcuts, and act accordingly.

## Shortcuts

These shortcuts are only available if `--type=auto` has been (implicitely) specified.

### string

If there are no special characters in the pattern, then a line will match if the pattern occurs anywhere on a line (as if `--type=contains` was specified).
```
# Show the lines containing "ne"
$ rak ne twenty
twenty
1:o𝐧𝐞
9:ni𝐧𝐞
19:ni𝐧𝐞teen
```

### §string

If the pattern starts with "`§`" (A7 SECTION SIGN), then the rest of the pattern must match as a **word** in a line.  So `rak §string` is the same as `rak string --type=words`.

> Depending on your keyboard and OS, the § character may be hard to type: if you're going to use this feature often, it probably makes sense to create a keyboard shortcut for it.

```
# Show the lines that contain "six" as a word
$ rak §six twenty
twenty
6:𝐬𝐢𝐱
```

### ^string

If the pattern starts with "`^`" (5E CIRCUMFLEX ACCENT), then the rest of the pattern must match at the **start** of a line.  So `rak ^string` is the same as `rak string --type=starts-with`.
```
# Show the lines starting with "e"
$ rak ^e twenty
twenty
8:𝐞ight
11:𝐞leven
18:𝐞ighteen
```

### string$

If the pattern ends with "`$`" (24 DOLLAR SIGN), then the rest of the pattern must match at the **end** of a line.  So `rak string$` is the same as `rak string --type=ends-with`.
```
# Show the lines ending with "o"
$ rak o$ twenty
twenty
2:tw𝐨
```

### ^string$

If the pattern starts with "`^`" (5E CIRCUMFLEX ACCENT) and ends with "`$`" (24 DOLLAR SIGN), then the rest of the pattern must match the line in its entirety.  So `rak ^string$` is the same as `rak string --type=equal`.
```
# Show all the lines that consist of "seven"
$ rak ^seven$ twenty
twenty
7:𝐬𝐞𝐯𝐞𝐧
```

### / regex /

If the pattern starts and ends with "`/`" (2F SOLIDUS), then the rest of the pattern will be interpreted as a **regex** to match lines with.  So `rak /string/` is the same as `rak string --type=regex`.  The pattern generally will need to be quoted as the shell may interpret the pattern in unwanted ways otherwise.
```
# Show all the lines with exactly on character between "e" and "t"
$ rak /e.t/ twenty
twenty
17:sev𝐞𝐧𝐭een
20:tw𝐞𝐧𝐭y
```

### { code }

If the pattern starts with "`{`" (7B LEFT CURLY BRACKET) and ends with "`}`" (7D RIGHT CURLY BRACKET), then the rest of the pattern will be interpreted as Raku code.  So `rak '{string}'` is the same as `rak string --type=code`.  The pattern generally will need to be quoted as the shell may interpret the pattern in unwanted ways otherwise.
```
# Show all lines starting with "t" titlecased
$ rak '{.tc if /^ t /}' twenty
twenty
2:Two
3:Three
10:Ten
12:Twelve
13:Thirteen
20:Twenty
```

### \*code

If the pattern starts with "`*`" (2A ASTERISK), then the rest of the pattern will be interpreted as Raku code.  So `rak *string` is the same as `rak string --type=code`.  The pattern generally will need to be quoted as the shell may interpret the pattern in unwanted ways otherwise.

This is typically used for short pieces of code that use [Whatever-currying](https://docs.raku.org/type/Whatever#index-entry-Whatever-currying)
```
# Reverse the order of the characters of each line
$ rak '*.flip' twenty
twenty
1:eno
2:owt
3:eerht
4:ruof
5:evif
6:xis
7:neves
8:thgie
9:enin
10:net
11:nevele
12:evlewt
13:neetriht
14:neetruof
15:neetfif
16:neetxis
17:neetneves
18:neethgie
19:neetenin
20:ytnewt
```

## On Raku regexes

Everybody knows regular expressions, right?  Well, I guess that's true to some extent.  But regular expressions haven't been regular for a long time already.  That's why it was decided to just call them ["regexes"](https://docs.raku.org/language/regexes) in the Raku Programming Language.

Regexes in Raku look like your average regular expression: some recipe to tell a state machine to look for matches.  The biggest difference with other regular expression implementations, is that Raku regexes are **code** under the hood that you specify with Raku's regex syntax.  You can think of a regex as a piece of code that you can call inside another regex, like a subroutine or a method.  This has all sorts of implications, but that will be for another series of blog posts.

Within the context of `rak`, the most important thing to know, is that you have to quote any non-alphanumeric character in Raku regexes, unless it has a special meaning in the context of regexes, such as `.`, `*`, `?`, to name but a few.  The reason for this strictness, is to ensure future compatibility of regexes, should another non-alphanumeric character get a special meaning in Raku regexes at some time in the future.
```
# Show all lines that have an "h" in them, optionally followed by a "t"
$ rak '/ h t? /' twenty
twenty
3:t𝐡ree
8:eig𝐡𝐭
13:t𝐡irteen
18:eig𝐡𝐭een
```
Note that in the above example we used `?` to indicate that the "t" may or may not occur.

You should also be aware that whitespace in Raku regexes has no meaning.  So whitespace will need to be explicitely specified if it is intended to be part of the regex.  However, if there is whitespace between alphanumeric characters, it will warn.
```
# Look for one, with unneeded whitespace
% rak '/ o ne /' twenty
Potential difficulties:
    Space is not significant here; please use quotes or :s (:sigspace) modifier (or, to suppress this warning, omit the space, or otherwise change the spacing)
    ------> / o⏏ ne /
twenty
1:𝐨𝐧𝐞
```

The pipe character `|` can be use to indicate alternatives:
```
# Look for lines with either "foo" or "bar"
$ rak '/ one | four /' twenty
twenty
1:𝐨𝐧𝐞
4:𝐟𝐨𝐮𝐫
14:𝐟𝐨𝐮𝐫teen
```

The `<<` marker indicates the start of a word, and `\w*` indicates zero or more alphanumeric characters.
```
# Look for lines in which there are words that start with "fi"
$ rak '/ << fi \w+ /' twenty
twenty
5:𝐟𝐢𝐯𝐞
15:𝐟𝐢𝐟𝐭𝐞𝐞𝐧
```

There's a lot more about [regexes](https://docs.raku.org/language/regexes) that can be told.  There is even a dedicated book about it by *Moritz Lenz*: [Parsing with Regexes and Grammars: A Recursive Descent into Parsing](https://www.amazon.com/Parsing-Perl-Regexes-Grammars-Recursive/dp/1484232275#customerReviews).

## Conclusion

This concludes part 3 of a series of blog posts about `rak`.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
