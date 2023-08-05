# Phasers (Part 1 of 2)
The following two blog posts will look at the special blocks in Perl, such as `BEGIN` and `END`, and the possibly subtle change in semantics with so-called phasers in Raku.

Raku has generalised some Perl features as "phasers" that weren't covered by special blocks in Perl. And it has added other phasers that are not covered by any (standard) Perl functionality at all.

One important feature to remember about phasers is that they are not part of the normal flow of a program's execution. The runtime executor decides when a phaser is being run depending on the type of phaser and context. Therefore, all phasers in Raku are spelled in uppercase characters to make them stand out.

## An overview
Let's start with an overview of Perl's special blocks and their Raku counterparts in the order they are executed in Perl:
```
  Perl        Raku     Notes
  -----------------------------------------------------------
  BEGIN       BEGIN    Not run when loading pre-compiled code
  UNITCHECK   CHECK
  CHECK                No equivalent in Raku
  INIT        INIT
  END         END
  -----------------------------------------------------------
```
These phasers in Raku are usually called program "execution phasers" because they are related to the execution of a complete program, *regardless* of where they are located in a program.

## BEGIN
The semantics of the `BEGIN` special block in Perl and the `BEGIN` phaser in Raku are the same. It specifies a piece of code to be executed immediately, as soon as it has been parsed (so before the program, aka the compilation unit, as a whole has been parsed).

There is, however, a caveat with the use of `BEGIN` in Raku: modules in Raku are pre-compiled by default, as opposed to Perl which does not have any pre-compilation of modules or scripts.

As a user or developer of Raku modules, you do not have to think about whether a module should be pre-compiled (again) or not. This is all done automatically under the hood when installing a module and after each Rakudo Raku update. It is also done automatically whenever a developer makes a change to a module. The only thing you might notice is a small delay when loading a module.

This means that the `BEGIN` block is executed **only** when the pre-compilation occurs, *not* every time the module is loaded. This is different from Perl, where modules ordinarily exist only as source code that is compiled whenever a module is loaded (even though that module can load already compiled native library components).

This may cause some unpleasant surprises when porting code from Perl to Raku, because pre-compilation may have happened a long time ago or even on a different machine (if it was installed from an OS-distributed package). Consider the case of using the value of an environment variable to enable debugging.

In Perl you could write this as:
```
# Perl
my $DEBUG;
BEGIN { $DEBUG = $ENV{DEBUG} // 0 }
```
This would work fine in Perl, as the module is compiled every time it is loaded, so the `BEGIN` block runs every time the module is loaded. And the value of `$DEBUG` will be correct, depending on the setting of the environment variable.

But not so in Raku. Because the `BEGIN` phaser is executed only once, when pre-compiling, the `$DEBUG` variable will have the value determined at module pre-compilation time, *not* at module-loading time!

An easy workaround would be to inhibit pre-compilation of a module in Raku:
```
# Raku
# this compilation unit should not be pre-compiled
no precompilation;
```
However, pre-compilation has several advantages that you don't want to dismiss easily:
- Data structure setup has to be done just once. If you have data structures that must be set up each time a module is loaded, you can do it once when a module is pre-compiled. This may be a huge time- and CPU saver if the module is loaded often.
- It can load modules much faster. Because it doesn't need to parse any source code, a pre-compiled module loads much faster than one that's compiled over and over again. A prime example is the core setting of Rakudo — the part that is written in Raku. This consists of a 68 KLOC/2.3MB source files (generated from many separate source files for maintainability). It takes about a minute to compile this source file during Rakudo installation. It takes about 125 milliseconds to load this pre-compiled code at Rakudo startup. This is almost a **500x** speed boost!

Some other features of Perl and Raku that implicitly use `BEGIN` functionality have the same caveat. Take this example where we want a constant `DEBUG` to have either the value of the environment variable `DEBUG` or, if that is not available, the value 0:
```
# Perl
use constant DEBUG => $ENV{DEBUG} // 0;
```
```
# Raku
my constant DEBUG = %*ENV<DEBUG> // 0;
```
The best equivalent in Raku is to use an `INIT` phaser:
```
# Raku
# sigilless variable bound to value
INIT my \DEBUG = %*ENV<DEBUG> // 0;
```
As in Perl, the `INIT` phaser is run just before execution starts.

You can also use Raku's module pre-compilation behaviour as a feature:
```
# Raku
say "This module was compiled at { BEGIN DateTime.now }";
# This module was compiled at 2023-08-05T17:23:24.837865+02:00
```
But more about that syntax later.

## UNITCHECK
The `UNITCHECK` special block's functionality in Perl is performed by the `CHECK` phaser in Raku. Otherwise, it is the same; it specifies a piece of code to be executed when parsing of the current compilation unit is done.

## CHECK
There is **no** equivalent in Raku of the Perl `CHECK` special block. The main reason is you probably shouldn't be using the `CHECK` special block in Perl anymore; use `UNITCHECK` instead because its semantics are much saner. (It's been available since Perl version 5.10!)

## INIT
The functionality of the `INIT` phaser in Raku is the same as the `INIT` special block in Perl. It specifies a piece of code to be executed just before the code in the compilation unit is executed.

In pre-compiled modules in Raku, the `INIT` phaser can serve as an alternative to the `BEGIN` phaser.

## END
The `END` phaser's functionality in Raku is the same as the `END` special block's in Perl. It specifies a piece of code to be executed *after* all the code in the compilation unit has been executed or when the code decides to exit (either intended or unintended because an exception is thrown).

## An example
Here's an example using all four program execution phasers and their Perl special block counterparts:
```
# Perl
say "running in Perl";
END       { say "ran END"   }
INIT      { say "ran INIT"  }
UNITCHECK { say "ran UNITCHECK" }
BEGIN     { say "ran BEGIN" }
# ran BEGIN
# ran UNITCHECK
# ran INIT
# running in Perl
# ran END
```
And the same in Raku:
```
# Raku
say "running in Raku";
END   { say "ran END"   }
INIT  { say "ran INIT"  }
CHECK { say "ran CHECK" }
BEGIN { say "ran BEGIN" }
# ran BEGIN
# ran CHECK
# ran INIT
# running in Raku
# ran END
```

## Summary
All of the special blocks still in use in Perl, have a counterpart in the Raku Programming Language.  However, due to module pre-compilation, the semantics of the `BEGIN` phaser is slightly different.  In some cases it is then better to use the `INIT` phaser instead.
