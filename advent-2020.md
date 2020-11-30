Helping the Github Action elves
===============================

As a [Raku Programming Language](https://raku.org) module developer, you are sometimes surprised by the tools that you use.  In this case, yours truly was surprised by a recent update of the excellent [App::Mi6](https://modules.raku.org/dist/App::Mi6:cpan:SKAJI) tool by *Shoichi Kaji*.  After an upgrade, it started adding a `.github/workflows/test.yml` file to new distributions.  And this in turn caused Github to test the distribution after each commit using [Github Actions](https://github.com/features/actions).  Which is great, especially if it finds problems!

So what did this file consist of?

```yaml
    name: test

    on:
      push:
        branches:
          - '*'
        tags-ignore:
          - '*'
      pull_request:

    jobs:
      raku:
        strategy:
          matrix:
            os:
              - ubuntu-latest
              - macOS-latest
              - windows-latest
            raku-version:
              - 'latest'
        runs-on: ${{ matrix.os }}
        steps:
          - uses: actions/checkout@v2
          - uses: Raku/setup-raku@v1
            with:
              raku-version: ${{ matrix.raku-version }}
          - name: Install Dependencies
            run: zef install --/test --test-depends --deps-only .
          - name: Install App::Prove6
            run: zef install --/test App::Prove6
          - name: Run Tests
            run: prove6 -l t
```

Quite a lot of [YAML](https://en.wikipedia.org/wiki/YAML) in there!  But the gist is basically clear: run the tests of this module on the latest [Ubuntu](https://en.wikipedia.org/wiki/Ubuntu) / [MacOS](https://en.wikipedia.org/wiki/MacOS) / [Windows](https://en.wikipedia.org/wiki/Microsoft_Windows) operating systems, and use the latest Raku version for that.  It was really great to see how easy it was to automatically get [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) support for your module.

And of course, as a module developer, you will get a notice (an email in this case) if testing of a module would fail.  That's how I found out that many modules that rely on [`NativeCall`-calls](https://docs.raku.org/language/nativecall) to [C-library](https://en.wikipedia.org/wiki/C_standard_library) functions that depend on [POSIX](https://en.wikipedia.org/wiki/POSIX) semantics, will simply fail on Windows.

Loving the green lights
-----------------------
It's always great to see green, like in the test results of [Hash::LRU](https://github.com/lizmat/Hash-LRU/actions/runs/337667222).  But something struck yours truly: more than **5 minutes** for testing?  Looking at the [timing of each step](https://github.com/lizmat/Hash-LRU/runs/1331938416?check_suite_focus=true) shows that the "Install [`App::Prove6`](https://modules.raku.org/dist/App::Prove6)" step took **over 3 minutes**!  While the actual tests of the module only ran for **two seconds**.  It looked like there was a lot of overhead involved, especially if a module did not have any external dependencies.

Now, when I am testing out modules locally, I usually do it like this:

```raku
    raku -Ilib t/01-basic.t  # or whatever test-file that shows a problem
```

Why?  Well, really because this allows one to directly add any debugging code to the test-file in case of failures, to more easily track the bug down.  And if there's an execution error, `--ll-exception` usually gets added to the call as well to get a more revealing backtrace, like so:


```raku
    raku -Ilib --ll-exception t/01-basic.t  # make sure we get a *full* backtrace
```

However, if Continuous Integration testing comes up with an execution error, then you don't get a full backtrace usually, which often does not help tracing the issue.  Especially if you cannot reproduce the problem locally.

Making things faster, better and more economic
----------------------------------------------
So, why not embed this manual workflow in a nice script, and add that to the distribution?  And make sure that only that script gets run?  That seems like an easy idea to implement.  And it was!  The script (called [`run-tests`](https://github.com/lizmat/Hash-LRU/blob/master/run-tests)) basically became (slightly shortened for this blog post):

```raku
    my @failed;
    my $done = 0;

    for "t".IO.dir(:test(*.ends-with: '.t' | '.rakutest')).map(*.Str).sort {
        say "=== $_";
        my $proc = run "raku", "--ll-exception", "-Ilib", $_, :out, :merge;
        if $proc {
            $proc.out.slurp;
        }
        else {
            @failed.push($_);
            if $proc.out.slurp -> $output {
                say $output;
            }
            else {
                say "No output received, exit-code $proc.exitcode()";
            }
        }
        $done++;
    }

    if @failed {
        say "FAILED: {+@failed} of $done:";
        say "  $_" for @failed;
        exit +@failed;
    }

    say "\nALL {"$done " if $done > 1}OK";
```

And the final 6 lines of the YAML file were changed to:

```yaml
      - name: Run Tests
        run: raku run-tests
```

Total timing of testing the [Hash::LRU](https://github.com/lizmat/Hash-LRU) module typically dropped to [below one minute](https://github.com/lizmat/Hash-LRU/actions/runs/375191322).  That's a lot of time and CPU cycles saved!  Of course, this will be less if there are dependencies that need to be installed.  But the shorter turn-around time, as well as seeing complete backtraces if something goes wrong, have definitely helped yours truly!  And as a bonus, testing locally has now also become easier and cleaner, especially if all goes well:

```text
    Welcome to Rakudo(tm) v2020.11.
    Implementing the Raku(tm) programming language v6.d.
    Built on MoarVM version 2020.11.

    Testing Hash-LRU
    === t/01-basic.t

    ALL OK
```

So should you copy this test-file to your distribution and adapt the Github Actions YAML accordingly?  Perhaps.  But perhaps it is better to find out what works best for *you* as module developer.  And build on this idea.  Perhaps start by copying the `run-tests` script and adapt it to your liking.  Whatever works best!

New to Raku?
------------

If you're new to Raku, you might appreciate some explanation of what the `run-tests` script actually does.  So here goes:

```raku
    my @failed;
    my $done = 0;
```

Sets up an array `@failed` for keeping the names of test-files that failed somehow, and a `$done` counter for the number of test-files that were done.

```raku
    for "t".IO.dir(:test(*.ends-with: '.t' | '.rakutest')).map(*.Str).sort {
```

This may be the hardest to grok if you're new to Raku.  What it does, is that it basically looks into the "t" directory `"t".IO` then starts looking for files `.dir(` that either have the ".t" or ".rakutest" extension `:test(*.ends-with: '.t' | '.rakutest'))`, change the resulting `IO::Path` objects to strings `.map(*.Str)`, then `.sort` these and loop over them `for ... {`.

```raku
        say "=== $_";
        my $proc = run "raku", "--ll-exception", "-Ilib", $_, :out, :merge;
```

Show which test-file is being tested `say "=== $_";` and run the actual actual test file `run "raku", "--ll-exception","-Ilib", $_,` and make sure that its STDOUT and STDERR output become available as a single stream `:out, :merge;` and put the resulting `Proc` object into `my $proc`.

```raku
        if $proc {
            $proc.out.slurp;
        }
```

If the run of the test file was successful `if $proc`, then simply eat all output and don't do anything with it `$proc.out.slurp`.

```raku
        else {
            @failed.push($_);
            if $proc.out.slurp -> $output {
                say $output;
            }
            else {
                say "No output received, exit-code $proc.exitcode()";
            }
        }
```

If not successful `else`, then add the name of the failed test file to the list of failed tests `@failed.push($_)`.  If there was any output `if $proc.out.slurp`, store it in a variable `-> $output` and show it to the world `say $output`.  If there was no output `else`, let the world know there was none with the exitcode `say "No output received, exit-code $proc.exitcode()"`.

```raku
        $done++;
    }
```

Remember that we've done a test-file, regardless of whether successful or not `$done++`.

```raku
    if @failed {
        say "FAILED: {+@failed} of $done:";
        say "  $_" for @failed;
        exit +@failed;
    }
```

If there was any test-file that failed `if @failed`, tell the world how many failed `say "FAILED: {+@failed} of $done:"` and show the names of the test-files that failed `say "  $_" for @failed`, and then exit the script indicating an error state `exit +@failed` in concordance with the [TAP-protocol](https://en.wikipedia.org/wiki/Test_Anything_Protocol).

```raku
    say "\nALL {"$done " if $done > 1}OK";
```

If we made it here, it's been all ok, so show that with the number of files, but *only* if it is more than one `"$done " if $done > 1`.

Some more information on the Raku features used in this program:

- [`.IO`](https://docs.raku.org/routine/IO#(Cool)_method_IO)
- [`.dir`](https://docs.raku.org/type/IO::Path#routine_dir)
- [`.map`](https://docs.raku.org/type/Any#routine_map)
- [`*.Str`](https://docs.raku.org/type/Whatever)
- [`.sort`](https://docs.raku.org/type/Any#method_sort)
- [`run`](https://docs.raku.org/language/independent-routines#sub_run)
- [`Proc`](https://docs.raku.org/type/Proc)
- [`exit`](https://docs.raku.org/language/independent-routines#sub_exit)

Conclusion
----------

With a little bit of work, you can make it easier for yourself *and* the Github Action elves.  And be more considerate of the environment as well, as too many elves working too hard is not good for the environment!
