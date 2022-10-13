# It's time to rak! (Part 2)

This is a follow-up on an [introduction to rak](https://dev.to/lizmat/its-time-to-rak-part-1-30ji).  This blog post builds on that.

But before discussing all of the myriad options of `rak`, it is important to know how the specification of options work exactly, and how they interact with any of *your* customized options.

## One dash vs two dash

If an option is specified by a single dash, then all of its letters are considered to be single letter options.  So `-im` is really short for `--i --m`.  Since `rak` by itself does not not have any single letter options, this implies that single letter options are always custom options.

However, you do *not* have to give your custom options a single letter.  Especially if the option is not a boolean flag, it is advisable to give your custom option more than one letter (it will make life easier for you, mnemonically speaking).

## Specifying boolean flags

Any option that starts with one or two dashes, and which is **not** followed by a equal sign (`=`), is considered to be a boolean flag, by default indicating *True*.  The default meaning of a boolean flag can be reversed in two ways:

- --/foo    *a single slash after the dashe(s) indicates* **False**
- --no-foo  *the string `no-` after the initial dashe(s) indicates* **False**

Specifying an option with a *False* value is typically needed if the default for that option is *True*.

## Specifying other values

Any option that needs a value other than a boolean, must specify this with an equal sign (`=`), followed by a value.  Whether or not that value needs to be quoted, and how they are to be quoted, really depends on the shell that you use to access `rak`.  In the following examples, quoting with single quotes is assumed to be the way to do that.  Some shells (most notably on Windows) require double quotes instead (so YMMV).

## Some basic custom options

Let's create some basic shortcuts that you will probably need the most.  The `--save` option is special: it disables all of the other options (after having them checked for validity).  It takes 

## Setting up default options

## Changing a custom option

## Removing a custom option

## Conclusion

This concludes part 2 of a series of blog posts about `rak`.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
