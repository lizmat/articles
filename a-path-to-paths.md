# A path to paths
The other day, not one but two people tried to use my [`rak`](https://raku.land/zef:lizmat/rak) module to create a custom file system search utility.  And had problems getting it to work.

Now, truth be told: they were the first people to use that module other than myself for the [`App::Rak`](https://raku.land/zef:lizmat/App::Rak) command line interface (as described in [It's time to rak!](https://dev.to/lizmat/its-time-to-rak-part-1-30ji).  And apparently, the use of the plumbing of `App::Rak` was less straightforward than I expected, specifically with regards to the way results are returned.

It wasn't until a bit later that I realized they were reaching for the wrong tool.  What they really wanted was to apply some search criteria to a list of files, as determined by some simple rules.

## paths
Well, there's a module for that: [`paths`](https://raku.land/zef:lizmat/paths), a fast recursive file / directory finder.  Which is one of the dependencies of `rak`.

All of the code examples here assume you have also done a `use paths` in your code.  And if the `paths` module is not installed yet, you should install it with `zef install paths` in your shell.

So how do you use it?
```raku
.say for paths;
```
will produce a list of **all** files from the currenty directory, **except** the ones that reside in a directory that starts with a period (so, e.g. a `.git` directory would be skipped).

So, what it you would like to get all JSON-files (as defined by their `.json` extension?
```raku
.say for paths(:file(*.ends-with(".json")));
```
What if you'd like to list all files in ".git" directories?
```raku
.say for paths(:dir(".git"));
```
Or you're just interested in directory names, not in the files inside directories?
```raku
.say for paths(:!file);
```
The `:!file` indicates that you're not interested in files.  This is Raku's way of specifying the named argument "file" with a `False` value.  Some would write this as `file => False`, which would also work in Raku.

All of the above examples assumed the current directory as a starting place.  The `paths` subroutine also takes an optional positional parameter: the directory from which to start.  So if you want to know all of the directories on your computer, you could start from the root directory:
```raku
.say for paths("/", :!file);
```
This may take a while!

## Paths as strings
The Raku Programming Language has the concept of an [`IO::Path`](https://docs.raku.org/type/IO/Path) object, which conceptually consists of a volume, a directory, and a basename. It supports both purely textual operations, and operations that access the filesystem, e.g. to resolve a path, or to read all the content of a file.

Unfortunately, creating such an object is relatively expensive, so `paths` has chosen to just provide absolute paths as strings.  If you want to work with `IO::Path` objects, the only thing that needs to be done, is to call the `.IO` method on the path.

For instance, if you would like to know the names of the files that contain the word "frobnicate", you could do:
```raku
.say for paths.grep: *.IO.slurp.contains("frobnicate");
```
The [`.IO`](https://docs.raku.org/type/Cool#method_IO) turns the path string into an `IO::Path` object, the [`.slurp`](https://docs.raku.org/type/IO/Path#routine_slurp) reads the whole contents of the file into memory as a string assuming UTF-8 encoding, and the [`.contains`](https://docs.raku.org/type/Str#method_contains) returns `True` if the given string was found in its invocant.

Now, if you do that, there's a good chance that this will end in an execution error, something like `Malformed UTF-8 near byte 8b at line 1 col 2`.  That's because there's a good chance that at least one of the files is a binary file.  Which is generally **not** valid UTF-8.

You could just ignore those cases with:
```
.say for paths.grep: { .contains("frobnicate") with .IO.slurp }
```
The `slurp` method will return a [`Failure`](https://docs.raku.org/type/Failure) if it couldn't complete the reading of the file.  The `with` then will only topicalize the value if it got something defined (and `Failure`s are considered to **not** be defined in this context).  Then the `contains` method is called as before and we get either `True` or `False` from that.

But doing it this way may just be a little expensive resource wise.  If resource usage is an issue for your program, then maybe there's a better way to find out whether something contains text or binary information.  And there is: with the sister module [`path-utils`](https://raku.land/zef:lizmat/path-utils).

## Path utilities

The `path-utils` module contains **41** subroutines that take a path string and then perform some check on that path.  Let's look at `path-is-text`: "Returns 1 if path looks like it contains text, 0 if not".
```raku
use path-utils <path-is-text>;

.say for paths.grep: { path-is-text($_) && .IO.slurp.contains("frobnicate") }
```
But what if you'd only like to look up texts in PDF files?  Well, the selection part can be done efficiently by `path-utils` as well, with the `path-is-pdf` subroutine.
```raku
use path-utils <path-is-pdf>;

.say for paths.grep: { path-is-pdf($_) }
```
but that would only show the files that appear to be PDF files.  To actually search in them, you could e.g. instance use *Steve Roe*'s [`PDF::Extract`](https://raku.land/zef:librasteve/PDF::Extract) module.
```raku
use path-utils <path-is-pdf>;
use PDF::Extract;

.say for paths.grep: { path-is-pdf($_) && Extract.new(:file($_)).text.contains("frobnicate") }
```

## Conclusion
It is always important to really understand the question, and to ask further if you don't understand the question, or the question askers do not understand your reply.

In this case, pointing these two Raku developers to the `paths` module, made their project suddenly (almost) a piece of cake.

And for me, it was a fine reason to highlight these cool modules in the [Raku ecosystem](https://raku.land)!
