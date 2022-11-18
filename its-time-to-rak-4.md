# It's time to rak! (Part 4)

This blog post will discuss the ways you can specify *where* to look for matches with [`rak`](https://raku.land/zef:lizmat/App::Rak) as part 4 of the [It's time to rak!](https://dev.to/lizmat/its-time-to-rak-part-1-30ji) blog series.

## From here on down

As we've seen in the earlier instalments, you can very easily search for a string using a literal string or a Raku regex in all files that look like they contain text.
```
# look for "foo" in all files
$ rak foo

# Search for "foo" in files of the "lib" directory
$ rak foo lib
```
And we've seen we can also limit the search to a single file:
```
# Look for "ve" anywhere on any line in file "twenty"
$ rak --type=contains ve twenty
```
These are three of the very basic ways to specify where to search: the first by **not** specifying anything, which implies all files in the current directory and any subdirectories of which the name does not start with a period.

The second basically being the same as the first, but starting from the "lib" directory on down, rather than from the current directory.

The third being the specification of a single file to look in.  And that single file does not need to exist on the local filesystem!  It can also be a URL.  Let's look for the word "reading" in the first blog post of this series:
```
$ rak §reading https://dev.to/lizmat/its-time-to-rak-part-1-30ji
https://dev.to/lizmat/its-time-to-rak-part-1-30ji
598:aria-label="Add to 𝐫𝐞𝐚𝐝𝐢𝐧𝐠 list"
954:<p>Thank you for 𝐫𝐞𝐚𝐝𝐢𝐧𝐠 all the way to the end!</p>
```
What this basically does is to download the indicated resource (courtesy of `curl`) into a temporary file, search in that while keeping the original URL as "the filename", and remove the file automatically when it's done.

## Actually only two

If you look at the above, then you realize that there are actually only two types of specification: a directory or a file (whether locally or remote).  And that a directory will be recursed into to look for files to include in the search.

The search for files in a directory, and its subdirectories, can be influenced by **40** different arguments.  This blog post will **not** mention all of them.  You can call:
```
# produce extensive help on filesystem filters
$ rak --help=filesystem --pager=less
```
to get a more in-depth description of the logic for each of the options should you need a feature that is not covered in this blog post.  The `--pager` argument is to let you more easily scroll the extensive text, but is of course not necessary.

What should be noted here is that these filesystem filters are **only** applied on subdirectories and files in those subdirectories.  So **not** on any directory or file that you specify *directly*.  With one caveat: many shells auto-expand anything you specify on the command line if they can: and these will be considered to be directly specified by `rak`, as it does not have a way to distinguish between what you typed and what the shell expanded it to.  For example:
```
# Search all files and all subdirectories
$ rak foo *
```
The `*` in the shell will effectively do `ls -d *`.  In practice, this is *almost* the same as not specifying anything at all.  But with one subtle difference: none of the filesystem filters will be applied to what the shell expanded to.  Whereas if you would not specifying anything (or `.` to indicate the current directory), the filesystem filters **would** be applied, because you (implicitely) specified only the current directory.  So only the current directory would be exempt from filesystem filters.

## At the base

The two most important filesystem filters are `--file` and `--dir`.  They expect a piece of code that will be given the [`basename`](https://docs.raku.org/type/IO::Path#method_basename) of a file or a directory, and which should return a trueish value to allow the file / directory to be accepted.  And they can also be specified as a flag: `--file` for unconditional acceptance, and `--/dir` for unconditional denial.

By default, `--file` and `dir='!.starts-with(".")'` are assumed.  Which effectively means, don't recurse into directories that start with a period, and accept all files in there.

To make it easier for you to specify files given by one or more extensions, you can use the `--extensions` argument:
```
# Only accept files with the .bat extension
$ rak foo --extensions=bat
```
As the name of the argument implies, you can specify multiple extensions, separated by comma's:
```
# Only accept files with the .bat or the .ps1 extension
$ rak foo --extensions=bat,ps1
```
It's also possible to only accept files *without* extension with the `--extensions` argument by just not specifying any actual extension:
```
# Only accept files without extension
$ rak foo --extensions=
```
You can also specify one of the predefined groups of extensions.  For instance, if you would like to only include Raku and Markdown files in your search, you can do:
```
# Only accept Raku and Markdown files
$ rak foo --extensions=#raku,#markdown
```
Note that the groups of extensions are prefixed with `#`.  To get an up-to-date list of extension groups:
```
# List all known extensions
# rak --list-known-extensions
```

If there is no argument specified related to the basename of the file (any of the above here), then the content of each file will be checked to see if it looks like it contains text and only included if it is actually deemed to contain text.

## More peripherally

The rest of the filesystem filter arguments can be roughly divided into the following groups: by content, epoch, owner / group, numeric meta value, external program and by attribute.  Again, you can see all of the needed information about these by doing:
```
# produce extensive help on filesystem filters
$ rak --help=filesystem
```
In any case, the end result of all of these filters is an internal list of files that will be checked for the pattern.  You could think of this list as the haystack, and the pattern as the needle, as it were.

## More on the haystack

Apart from specifying paths after the pattern, there is also a `--paths=path1,path2` argument.  This is supposed to contain a comma separated list of paths.  So these two invocations are equivalent:
```
# Search in the "lib" and "doc" directories
$ rak foo lib doc
$ rak foo --paths=lib,doc
```
The `--paths` argument allows you to save a set of paths with a shortcut (as we've seen in [Customizing your options](https://dev.to/lizmat/its-time-to-rak-part-2-18ha).

You can also store filenames and/or paths in a file, and specify that file to be taken as the haystack specification: the `--paths-from=filename` and `--files-from=filename` options.  Each line of the specified file, will be taken as either a file or path specification.  The difference in handling is that if a file is specified on a line with `--paths-from`, it is accepted.  If a directory is specified on a line with `--files-from`, then it will be ignored as not being a file.  And either of these take `-` to mean to read from STDIN.

For open source developers, the `--under-version-control` argument may be of use.  When used in a git repository, it will set up the haystack with all the files that are under version control.

More extensive help on these and other haystack arguments can be obtained by doing:
```
# produce extensive help on haystack specification
$ rak --help=haystack
```

## Twisting the haystack

There is one argument that converts the haystack into a list with the absolute paths of all the files in the haystack: `--find`.  It changes the list of targets into a target itself, if you will.  So instead of looking for the pattern in the contents of the files of the haystack, you'd be looking in the *names* of the files instead.
```
# Show all filenames that have "lib" in their name
$ rak --find lib
```

And if you just want a list of filenames, you can omit the pattern altogether:
```
# Show all filenames from current directory on down
$ rak --find
```
And what if you would just like to see the names of *directories* instead of files?  Well, that'd be only legal way to use the `--file` argument as a negator:
```
# Show all directory names fro current directory down
$ rak --find --/file
```

## Conclusion

This concludes part 4 of a series of blog posts about `rak`.  It shows how you can instruct rak where to look for matches, to create a haystack if you will.  By applying different acceptance rules for files and subdirectories, for instance by looking at extensions.  It also shows how you can twist the haystack to just show filenames or the names of directories.

If you have any comments, find bugs, have recommendations / ideas, please submit them as issues at the [App::Rak repository](https://github.com/lizmat/App-Rak/issues).  If you would like to have a more direct interaction, you can visit the [#raku-rak](https://web.libera.chat/?channel=#raku-rak) channel on [Libera.chat](https://libera.chat).

Thank you (again) for reading all the way to the end!
