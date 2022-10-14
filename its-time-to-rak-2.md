# It's time to rak! (Part 2)

This is a follow-up on an [It's time to rak (Part 1)](https://dev.to/lizmat/its-time-to-rak-part-1-30ji).  This blog post builds on that, so you should maybe read that first if you haven't already.

With `rak`, anything that starts with a dash (`-`) is considered to be an "option".  Anything else that you specify on the command line, is considered to be an "argumentxs".  It does *not* matter in which order options are specified.  The order of arguments **is** important: the first being the pattern, and the other arguments being the paths to search in.

But before discussing all of the myriad options of `rak` (shown to you when you do `rak --help`), it is important to know how the specification of options work exactly, and how they interact with any of *your* customized options.

## One dash vs two dash

If an option is specified by a single dash, then all of its letters are considered to be a separate single letter option.  So `-im` is really short for specifying the `i` and `m` option, as if you would specify `rak -i -m`.

Since `rak` by itself does not not have any single letter options, this implies that a single letter option is always a custom option.  But more about that later.  Single letter options are sometimes referred to as a "short option".

If an option is specified with two dashes, then all of the identifying letters after that, are considered to be the name of the option.  So `--foo` is specifying the `foo` option.  This is sometimes referred to as a "long option".

The order in which options (anything that starts with a dash (`-`)

## Boolean flags

Any option that starts with one or two dashes, and which is **not** followed by a equal sign (`=`), is considered to be a boolean flag.  If only a name is specified, the flag is considerd to be *True*.

The meaning of a boolean flag can be changed to *False* in two ways:

### --/foo

A single slash after the dash(es) indicates **False**.

### --no-foo

The string `no-` after the initial dash(es) indicates **False**.

Specifying an option with a *False* value is typically needed if the default for that option is *True*.  Some examples:
```
# produce results without any highlighting
$ rak foo --/highlight

# produce results as if piping to a file
$ rak foo --no-human
```

## Specifying other values

Any option that needs a value other than a boolean, must specify this with an equal sign (`=`), followed by a value.  Whether or not that value needs to be quoted, and how they are to be quoted, really depends on the shell that you use to access `rak`.  In the following examples, quoting with single quotes is assumed to be the way to do that.  Some shells (most notably on Windows) require double quotes instead (so YMMV).
```
# specify a literal pattern at the end of a line
$ rak --type=ends-with foo

# same, with a regular expression
$ rak '/ foo $/'
```

## Creating a custom option

The `rak` command line utility allows you to create your own custom options, shortcuts if you will.  If you're used to another search utility, you might want to use the same option names as that other search utility.  Or you could decide to just create custom options following your own logic.

Creating / changing / removing a custom option, is done with the `--save=name` option.  If that is specified, all of the other options are checked for validity by name, but do **not** perform any function.  Instead, they become the set of options to be associated with the "name".  And those options themselves, are either one of `rak`s standard options, or previously defined custom options.

Note that you can give your custom options a single letter, which is especially handy if the option it is referring to, is a flag.  If it is not referring to a flag, it is advisable to give your custom option more than one letter (it will make life easier for you, mnemonically speaking).

To help you (or others) to later understand what a custom option was intended to do, you can add a description with the `--description` option.  Either when creating the custom option, or at a later time.

## Some basic custom options

Let's create some basic shortcuts that you will probably need the most.
```
# save --ignorecase as -i, without description
$ rak --ignorecase --save=i

# save --ignoremark as -m, with description
$ rak --ignoremark --description='Ignore any accents' --save=m

# add / change description -i at a later time
$ rak --description='Do not care about case' --save=i

# look for literal string "foo", don't check case or accents
$ rak foo -im
```

If you would like to have the pattern saved as (part of) a custom options, you need to use the `--pattern` form of pattern specification.
```
# save looking for whitespace at end of a line as --wseol
$ rak --pattern='/ \s+ $/' --save=wseol

# search for trailing whitespace
$ rak --wseol
```

If you would like to have one or more paths to be part of a custom option, you would need to specify that with the `--paths` option.
```
# save searching in Rakudo's committed files as --rakudo
$ rak --paths='~/Github/rakudo' --under-version-control --save=rakudo

# search for 'sub min' in Rakudo's source 
$ rak 'sub min' --rakudo
```

## Accepting a value with a custom option

Some options require a value.  So you need to be able to specify a value with a custom option referring to an option that needs a value.  You can do this in two ways: by specifying an exclamation mark `!` (to indicate a value is required) or by specifying a default (between square brackets `[` `]`).  Some examples:

```
# save --after-context as -A, requiring a value
$ rak --after-context=! --save=A

# save --before-context as -B, requiring a value
$ rak --before-context=! --save=B

# save --context as -C, setting a default of 2
$ rak --context='[2]' --save=C

# search for "foo" and show two lines of context
$ rak foo -C

# search for "foo" and show 4 lines of context
$ rak foo -C=4

# same, without equal sign
$ rak foo -C4
```

Note that you can omit the equal sign (`=`) on single letter options that take a numerical value.

## Setting up default options

If you are not happy with the default settings of `rak`, you can set the options you would like to have applied automatically every time you start `rak`.  This is possible by saving those options as a custom option with the special name `(default)`.
```
# set up smartcase by default
$ rak --smartcase --save='(default)'
```
After having done this, all of your searches will be case-insensitive, provided that the iteral pattern that you provide does **not** contain any uppercase characters.

```
# look for "foo" case-insensitively
$ rak foo

# look for "Foo", while taking case into account
$ rak Foo
```

## Changing a custom option

You can change a custom option just by `--save`ing it again.  You would need to specify all options that you want to keep again.  This may be a bit of a bother with a custom option that activates many other options.  To know what the current custom options are, you can use the `--list-custom-options`:
```
$ rak list-custom-options
-i: Do not care about case
  --ignorecase
-m: Ignore any accents
  --ignoremark
```

And then it'd be a matter of copy-and-paste.

## Removing a custom option

Removing a custom option, is just saving it again *without* any options specified.
```
# remove the --frobnicate custom option
$ rak --save=frobnicate

# Check there's no "frobnicate" option anymore
$ rak --frobnicate
Regarding unexpected option --frobnicate, did you mean:
 --proximate: Grouping of matched lines?
Specify --help for an overview of available options.
```

## About the configuration file

By default, the configuration file is at `~/.rak-config.json`.  You can specify the location of the
configuration file by setting the `RAK_CONFIG` environment variable:
```
# start rak with configuration file at /usr/local/rak-config.json
$ RAK_CONFIG=/usr/local/rak-config.json rak foo
```

You can also use that environment variable to start rak *without* any custom options (or default options), by **not** specifying a value:
````
# start rak with *without* any custom options
$ RAK_CONFIG= rak foo
```

## Conclusion

This concludes part 2 of a series of blog posts about `rak`.  This episode shows you how to specify options, and how you can make your own custom options out of the many, many options that `rak` provides out of the box.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
