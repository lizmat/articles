# Not So Obvious Semantic Changes (Part 1 of 2)

Apart from the visible changes between Perl and Raku, there are a small number of semantic changes in code that otherwise is identical between Perl and Raku.  The following two blog posts elaborate on these potential gotcha’s.

## Everything is Lexical in Raku
This simple statement is easy to understand, but for the experienced Perl programmer, maybe not all implications are immediately clear.

For instance, if one uses a module in Perl that imports symbols, like `Test::More`, the imported symbols become available in the entire compilation unit:
```
# Perl
{                          # create a scope
    use Test::More;        # use module inside scope
    ok 1, “test passes”;   # ok 1 - test passes
}  # all of the imported subs are available outside:
ok 1, “test also passes”;  # ok 1 - test also passes
```
Now compare this to the situation in Raku:
```
# Raku
{                          # create a scope
    use Test;              # use module inside scope
    ok 1, “test passes”;   # ok 1 - test passes
}
```
Inside the scope, the `ok` sub is known, and can be executed.  However if we want to call the `ok` sub outside of the scope, then this will cause a compilation error as the sub is not known outside of the lexical scope:
```
# Raku
{                          # create a scope
    use Test;              # use module inside scope
}
ok 1, "This test passes";
===SORRY!=== Error while compiling …
Undeclared routine:
    ok used at line …
```
Now, this in itself could be considered a not so very useful feature.  Because in Perl, it is i**not** possible to have multiple versions of the same module installed at the same time (unless you start playing really nasty tricks with `-I` or `use lib`, which does not improve maintainability or stability).

In Raku however, it *is* possible to have multiple versions of the same module installed.  So suppose you have two versions of module `Foo` installed: version 1.21 and version 2.0.  Each of which exports a subroutine callewd `read`.  You can then provide easy access to either by localising the `use` statements inside separate subroutines:
```
# Raku
sub old-read($file) {
    use Foo:version<1.21>;
    read($file)
}
sub new-read($file) {
    use Foo:version<2.0>;
    read($file)
}
```
This feature is enormously powerful tool for version management of data.  And it makes life of system admins a lot easier, as you can freeze dependencies in the code, so that you never have to worry whether a running system will break by adding newer versions of already installed modules.

## Global Variables Vs. Dynamic Variables
Both Perl and Raku have a [`my`](https://docs.raku.org/routine/my) statement to define lexical variables.  Both Perl and Raku have [`our`](https://docs.raku.org/syntax/our) to define variables in a namespace that should be accessible from outside of that namespace.

So this works in Perl and Raku:
# Raku and Perl
package A {
    our $a = 42;
}
say $A::a;  # 42
```
However, global variables in a threaded environment such as Raku, are generally a bad idea.  Therefore, Raku has an intermediate for of variables that can be lexically defined, but which have global accessibility / discoverability: they are called "dynamic variables".

Dynamic variables can be recognised by the [`*`](https://docs.raku.org/language/variables#The_*_twigil) twigil (secondary sigil).  For instance, where Perl has `STDOUT` for the IO handle for standard output, Raku has `$*OUT`.

Perhaps this is better explained with an example: in Perl, `STDOUT` can be used as an object and you can call methods on it, e.g. `say`:
```
# Perl
STDOUT->say("hello world");  # hello world
```
You can do the same in Raku with `$*OUT`:
```
# Raku
$*OUT.say("hello world");    # hello world
```
Which is what the `say` command does under the hood.

So much for the similarities.  What if one wants to override standard output for a given scope?  Since `STDOUT` is a global variable in Perl, one cannot do much about this, other than using the `local` functionality of Perl.  But this is really not recommended.

In Raku, one defines a lexical with the same name and gives it a value:
```
# Raku
say "goodbye";               # goodbye
{
     my $*OUT = open("out",:w);  # STDOUT writes to file "out" from here
     say "cruel";                # written to file
}
say "world";                 # world
say "out".IO.slurp;          # cruel
```
Note that `say` command inside the scope sees the alternate version of `$*OUT`: the visibility is dynamic (looking up the execution stack for lexically defined dynamic variables), and therefore writes the string "cruel" to the file rather than showing it on standard output.

Almost all system variables in Raku are dynamic variables.  Let’s compare a few between Perl and Raku:
```
    description                  Perl            Raku
    ---------------------------------------------------
    standard input               STDIN           $*IN
    standard output              STDOUT          $*OUT
    standard error output        STDERR          $*ERR
    environment variable         %ENV            $*ENV
    raw command line arguments   @ARGV           @*ARGS
    current directory            cwd (use Cwd)   $*CWD
    current process ID           $$              $*PID
    module loading management    @INC            $*REPO
    name of executing program    $0              $*PROGRAM-NAME
    ---------------------------------------------------
```
Note that the dynamic variable of Raku are generally more descriptive (and longer) than the Perl equivalents.

## Summary
In this blog posts it was shown that everything in Raku is lexically scoped.  And that in Raku dynamic variables take the ecological niche that global variables occupy in Perl.
