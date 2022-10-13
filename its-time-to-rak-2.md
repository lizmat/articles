# It's time to rak! (Part 2)

This is a follow-up on an [introduction to rak](https://dev.to/lizmat/its-time-to-rak-part-1-30ji).  This blog post builds on that.

But before discussing all of the myriad options of `rak`, it is important to know how the specification of options work exactly, and how they interact with any of *your* customized options.

## One dash vs two dash

If an option is specified by a single dash, then all of its letters are considered to be single letter options.  So `-im` is really short for `-i -m`.  Since `rak` by itself does not not have any single letter options, this implies that single letter options are always custom options.

However, you do *not* have to give your custom options a single letter.  Especially if the option is not a boolean flag, it is advisable to give your custom option more than one letter (it will make life easier for you, mnemonically speaking).

## Specifying boolean flags

Any option that starts with one or two dashes, and which is **not** followed by a equal sign (`=`), is considered to be a boolean flag, by default indicating *True*.  The default meaning of a boolean flag can be reversed in two ways:

### --/foo

A single slash after the dashe(s) indicates **False**.

### --no-foo

The string `no-` after the initial dashe(s) indicates **False**.

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
$ rak foo --type=ends-with

# same, with a regular expression
$ rak '/ foo $/'
```

## Creating a custom option

The `rak` command line utility allows you to create your own custom options, shortcuts if you will.  If you're used to another search utility, you might want to use the same option names as that other search utility.  Or you could decide to just create custom options following your own logic.

Creating / changing / removing a custom option, is done with the `--save=name` option.  If that is specified, all of the other options are checked for validity by name, but do **not** perform any function.  Instead, they become the set of options to be associated with the "name".  And those options themselves, are either one of `rak`s standard options, or previously defined custom options.

## Some basic custom options

Let's create some basic shortcuts that you will probably need the most.  The `--save` option is special: it disables all of the other options (after having them checked for validity).  It takes the value as the name under which all of the specified options should be saved.

If you would live to have the pattern saved as (part of) a custom options, you need to use the `--pattern` form of pattern specification.
```
# save --ignorecase as -i, without description
$ rak --ignorecase --save=i

# save --ignoremark as -m, with description
$ rak --ignoremark --description='Ignore any accents' --save=m

# add / change description -i at a later time
$ rak --description='Do not care about case' --save=i
```

## Setting up default options

If you are not happy with the default settings of `rak`, you can set the options you would like to have applied automatilly every time you start `rak`.  This is possible by saving these options as a custom option with the special name `(default)`.
```
# set up smartcase by default
$ rak --smartcase --save='(default)'
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

## Removing a custom option

Removing a custom option, is just saving it again *without* any options.
```
# remove the --frobnicate custom option
$ rak --save=frobnicate
```

## Conclusion

This concludes part 2 of a series of blog posts about `rak`.  This episode showed you how to specify options, and how you can make your own custom options out of the many, many options that `rak` provides out of the box.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
