# It's time to rak! (Part 3)

This blog post will discuss the types of patterns you can specify with [`rak`](https://raku.land/zef:lizmat/App::Rak).

To start right off the bat: the `--type` argument indicates how a given pattern (specified as a string on the command line) should be interpreted.  Having to specify `--type=foo` all of the time, can be bothersome.  So there are a number of shortcuts that you can use to make your life easier.  These will be discussed at the end.

The results in all these examples, are based on the existence of a file called "twenty" in the current directory, which contains the words "one", "two", "three" ... "twenty", each on a separate line.  Because the contents of the line matches the line number, it hopefully makes understanding the example output in this post easier.

## --type=contains

Match all lines that contain the literal pattern **anywhere**.
```
# Look for "ve" anywhere on any line in file "twenty"
$ rak --type=contains ve twenty
twenty
5:fi𝐯𝐞
7:se𝐯𝐞n
11:ele𝐯𝐞n
12:twel𝐯𝐞
16:se𝐯𝐞nteen
```
Note that first the filename is shown, and then the actual matches with the pattern highlighted in the result.  This is *on* by default, if `rak` is being run by a human (aka, there's actually someone at the keyboard).  Or more technically, if STDIN is connected to a TTY.

Also note that the matched lines are prefixed with their line number (which happens to be the same as the textual version of the number).

## --type=words

Match any line that contains the literal pattern as a **word**.  A word here is defined as a string consisting of alphanumeric characters, with non-alphanumeric characters (or the absence of a character) on both ends.
```
# Look for "six" as a word on any line in file "twenty"
$ rak six twenty --type=words
twenty
6:**six**
```

In this case "six" was acceptable, because there are no characters before or after on the line.  So in that sense, it is a bad example of using "words" as a type of search.  Note that "sixteen" was *not* matched, because there was no word-boundary at "x".

## --type=starts-with

Match any line that **starts** with the literal pattern.
```
# Look for "seven" at the start of all lines in file "twenty"
$ rak seven twenty --type=starts-with
twenty
7:**seven**
16:**seven**teen
```

Note that in this case the line with "seventeen" *was* matched, because it starts with "seven".

## --type=ends-with

Match any line that **ends** with the literal pattern.
```
# Look for "ve" at the end of all lines in file "twenty"
$ rak ve twenty --type=ends-with
twenty
5:fi**ve**
12:twel**ve**
```

Note that in this case the lines with "seven", "eleven" and "seventeen" were *not* matched, because they didn't end with "ve", even though they had the literal string in them.

## --type=equal

Match any line that matches in its **entirety**  with the literal pattern.
```
% rak eight twenty --type=equal
twenty
8:**eight**
```

Note that in this case the line with "eighteen" was *not* matched, because it had additional characters after "eight".

## --type=regex

## Ignoring case and accents

## --type=code

## --type=auto

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
