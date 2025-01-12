# Towards more coverage

It was sometime in November (or was it December?) last year that my attention was drawn again to a feature of [`MoarVM`](https://moarvm.org) that is sadly under-documented and its existence is really only visible if you run `moar` without any arguments:
```
$ moar
ERROR: Missing input file.

USAGE: moar [--crash] [--libpath=...] input.moarvm [program args]
# more lines
  MVM_COVERAGE_LOG  Append (de-duped by default) line-by-line coverage messages to this file
# still more lines
```
Very terse documentation about a very useful feature to have in a virtual machine: the creation of a so-called "coverage log", that shows *which* lines of source code have been executed during the run of a process.  Such information can be used to find out whether test-files of a distribution actually test all of the possible code-paths in a distribution.

So how does such a coverage log look like?  Well, it's very easy to create one from the command line:
```
$ MVM_COVERAGE_LOG=log raku -e ''
```
That's it.  The above will create a file named "log" of about 500K bytes, with 9600+ lines.  It looks like this:
```
HIT  src/vm/moar/ModuleLoader.nqp  1
HIT  src/vm/moar/ModuleLoader.nqp  1
HIT  src/vm/moar/ModuleLoader.nqp  105
HIT  src/vm/moar/ModuleLoader.nqp  181
HIT  src/vm/moar/ModuleLoader.nqp  253
*
* many more lines
*
HIT  SETTING::src/core.c/Rakudo/Internals.rakumod  1803
HIT  SETTING::src/core.c/Rakudo/Internals.rakumod  1805
HIT  SETTING::src/core.c/Rakudo/Internals.rakumod  1786
HIT  SETTING::src/core.c/Rakudo/Internals.rakumod  1784
HIT  src/main.nqp  85
```
As you can see, that's a **lot** of lines for what is essentially a null-program.  But it tells you (almost) exactly what happened inside the machine.

That moment I decided to have an easy way to find out whether the tests of a module actually cover *all* of the possible code paths of that module.  Or at least find out how much was **not** covered by tests.  And if not covered, where the parts of the code are that were not covered by tests.

> The fact that I have [200+ modules](https://raku.land/zef:lizmat) in the Raku ecosystem, also has to do with that desire.

I hoped that it would not be very difficult to turn that into useful information.

## Post XMas
Sometime after Xmas I started working on that.  Every now and then it was *not* a SMOP (Simple Matter Of Programming), but now I'm glad to be able to announce that a more or less viable product is now available: [`Test::Coverage`](https://raku.land/zef:lizmat/Test::Coverage).

>In the initial stages of this development, I found out that there actually had been a previous attempt at processing coverage information and processing that in a sensible way: [`App::RaCoCo`](https://raku.land/zef:atroxaper/App::RaCoCo) by *Mikhail Khorkov*.  `App::RaCoCo` takes the approach of needing an author initiated action (starting the `racoco` application).  I wanted to have something that would be part of testing, and would automatically inhibit release with e.g. [`App::Mi6`](https://raku.land/zef:skaji/App::Mi6) if the coverage would **not** meet certain prerequisite values.

## Test::Coverage
So now there's [`Test::Coverage`](https://raku.land/zef:lizmat/Test::Coverage).  You can install that with zef: `zef install Test::Coverage`.

Use this as a module developer by adding a `coverage.rakutest` file in a (possibly new) `xt` directory in the root directory of a distribution.

And then add these lines to it:
```raku
use Test::Coverage;

plan 2;

coverage-at-least 80; # percent

uncovered-at-most 10; # lines
```
and then run `raku -I. xt/coverage.rakutest` from the command line.

> Note that the values *80* and *10* are just arbitrary values that feel like a good start: at least 80% coverage, with a maximum of 10 lines not getting covered.

Your `coverage.rakutest` script will now execute **all** of the test files of a distribution that could be found in coverage mode, process that information, and then output something like this:
```
$ raku -I. xt/coverage.rakutest
1..2
# Failed test 'Coverage 55.10% >= 80%'
# at xt/coverage.rakutest line 5
not ok 2 - Uncovered 22 <= 10 lines
# Failed test 'Uncovered 22 <= 10 lines'
# at xt/coverage.rakutest line 7
```
Well, that's not really informative now is it?  Fortunately, there are also options to make this produce more information, provided by the `Test::Coverage` module.

### report
The first is the `report` subroutine that will produce a more verbose report, much like this case (for the [`Text::Mathematical`](https://raku.land/zef:lizmat/Text::MathematicalCase) module).  So, adding `report;` to the script, we get:
```
Welcome to Rakudo™ v2024.12.
Implementing the Raku® Programming Language v6.d.
Built on MoarVM version 2024.12.
Coverage Report of 2025-01-11 20:27:31

Text::MathematicalCase (55.10%):
  Missed 22 lines out of 49:
  18 48 53 64 82 101 104 105 106 107 109 110 111 113 118 119 121 166 168
  187 188 189

Produced by Test::Coverage (0.0.5)
```
As you can see, it shows some system information, the name of the module (`Text::MathematicalCase`), the percentage of lines that were covered by the test-files (`55.10%`), the number of lines that were deemed to be coverable (`49`) and the number of lines that were **not** covered (`22`).

> Finding out which lines in a source file are deemed "coverable", turned out to be a bit more involved as expected.  Obviously, comment lines and empty lines cannot be covered, but could e.g. an empty `}` on a line be covered or not?  A follow-up blog post will go into this process in more depth.

But more importantly, it shows the **line numbers** of the lines that were not covered by the tests.  Useful information, but maybe not handy enough yet for someone who'd be willing to improve tests.

## Raku coverage files
Fortunately, there is a subroutine that is also provided by `Test::Coverage` that you can add to the `coverage.rakutest` test script that will produce more information: `source-with-coverage`.

Adding that to your script will not show anything different from before, **but** it will create a `coverage` directory as a sibling to the `t` directory, and create a source-file in there at the same relative location as in `lib`, but with the `.rakucov` extension.  So in this case a `coverage/Text/MathematicalCase.rakucov` file.

> Since you probably do **not** want to put these files into git, it is probably wise to add `*.rakucov` to your `.gitignore` file.

Each of the lines in the coverage file matches the original source file, but it has two characters prefixed to it.  These can be:
- "* " - line was coverable, and covered
- "✱ " - line was covered, but was not originally marked as coverable
- "x " - line was coverable, but **not** covered
- "  " - line was not coverable, and not covered

So an example of a well tested subroutine from this source file would be:
```raku
* my sub trans(Str:D \string, Pair:D \mapper --> Str:D) {
✱     my @source     := string.NFD;
✱     my @haystack   := mapper.key;
✱     my @translated := mapper.value;

      my uint32 @result;
*     for @source -> int $needle {
*         with @haystack.first($needle, :k) {
*             @result.push(@translated[$_]);
          }
✱         else {
*             @result.push($needle);
          }
      }

*     Uni.new(@result).Str;
  }
```
No `x`es, so all that could be covered, *was* covered.

An example of incomplete coverage from the same file:
```raku
* sub EXPORT(*@args, *%_) {
*     if @args {
          my $imports := Map.new( |(EXPORT::all::{ @args.map: '&' ~ * }:p) );
x         if $imports != @args {
x             die "Text::MathematicalCase doesn't know how to export: "
x               ~ @args.grep( { !$imports{$_} } ).join(', ')
          }
          $imports
      }
      else {
*         Map.new('&mc' => EXPORT::all::<&mc>)
      }
  }
```
So apparently the case of garbage input into `use Text::MathematicalCase` is not being tested yet, because of the `x`'s in front of this error checking.

If you're lazy, you can change the values in the `coverage.rakutest` test file to values that appear more acceptable at this point in time.  This will allow the test-file to pass and not inhibit any ecosystem uploads with e.g. `App::Mi6` (as these also run the test-files in the `xt` directory).

So let's do that: change the `80` to `55`, and the `1` to `22`, and remove the `report`, and we get:
```
$ raku -I. xt/coverage.rakutest
1..2
ok 1 - Coverage 55.10% >= 55%
ok 2 - Uncovered 22 <= 22 lines
```
Alternately, you can add `todo` statements to mark the failing test as one that will need fixing.  This is possible because the `Test::Coverage` module also exports all of `Test`'s subroutines as well, so you don't need to do an additional `use Test` for that.
```raku
use Test::Coverage;

plan 2;

todo "needs more tests";
coverage-at-least 80; # percent

todo "needs more tests";
uncovered-at-most 10; # lines
```
This will make sure that the lack of coverage will still make it possible to do a release with `App::Mi6`.

## Conclusion
With the `Test::Coverage` module, every Raku module developer is able to add coverage testing to their distributions in a very simple manner.  And this does not need to affect anything in the normal workflow of the developer: the `Test::Coverage` module need only be installed on the author's computer.

This is the first post of a series: follow-up posts will get more into the development process, and possible future developments of this new module.  Stay tuned!
