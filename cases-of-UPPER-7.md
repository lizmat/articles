# Doc Mirages

> This is part seven in the ["Cases of UPPER"](https://dev.to/lizmat/series/35190) series of blog posts, describing the Raku syntax elements that are completely in UPPERCASE.

This part will discuss the final set of (non-)existing phasers in the [Raku Programming Language](https://raku.org).

## But first: --doc

The `--doc` command line argument to `raku` is described as:
```
--doc         extract documentation and print it as text
--doc=module  use Pod::To::[module] to render inline documentation
```
So what that basically does, is process the source of the program to be executed as documentation, and render that documentation in various forms.

Internally this is achieved by loading the `Pod::To::xxx` module (with "xxx" defaulting to "Text" in case of `--doc` without arguments).  It then installs an `INIT` phaser that basically does:
```raku
INIT {
    say Pod::To::xxx.render($=pod);
    exit;
}
```
Note that the `Pod::To::Text` module is  installed as part of the rakudo distribution (whether it should be considered part of the Raku Programming Language specificatio is yet unclear).

Other renderers such as [`Pod::To::HTML`](https://raku.land/zef:raku-community-modules/Pod::To::HTML), [`Pod::To::PDF`](https://raku.land/zef:dwarring/Pod::To::PDF) and [`Pod::To::Markdown`](https://raku.land/zef:raku-community-modules/Pod::To::Markdown) would have to be installed first from the ecosystem (e.g. `zef install Pod::To::Markdown`).

## DOC

There is no single [`DOC`](https://docs.raku.org/syntax/DOC) phaser, there are actually three of them:
- `DOC BEGIN` - `BEGIN` phaser if --doc is set
- `DOC CHECK` - `CHECK` phaser if --doc is set
- `DOC INIT` - `INIT` phaser if --doc is set

Another why of thinking about this, is that the `DOC` phaser could be thought of as a sort of statement prefix that will conditionally add the phaser.  In pseudo-code, for `DOC BEGIN`:
```
if --doc is set {
    BEGIN ...;
}
```
Note that if the `--doc` is **not** specified, the phasers will **not** be executed.  Observe the difference between:
```
$ raku -e 'DOC BEGIN say "begin"; DOC CHECK say "check"; DOC INIT say "init"'
```
not showing anything, while:
```
$ raku --doc -e 'DOC BEGIN say "begin"; DOC CHECK say "check"; DOC INIT say "init"'
begin
check
init
```
*will* execute the phasers and shows the output.

There's only one "but": even though the `DOC` phasers are only added if `--doc` is specified, they *must* contain valid Raku code to prevent errors:
```
$ raku -e 'DOC BEGIN +-+'            
===SORRY!=== Error while compiling -e
Prefix + requires an argument, but no valid term found.
Did you mean + to be an opening bracket for a declarator block?
at -e:1
```

So what use are they?  Looking at the ecosystem, there actually only appears to be **one** type of use that makes sense.  And that is functionality offered by the [`POD::EOD` module](https://raku.land/zef:raku-community-modules/Pod::EOD), which places [declarator documentation](https://docs.raku.org/language/pod#Declarator_blocks) at the end of the documentation, rather than where they occur in the source code.  Which works because it directly modifies the content of the `$=pod` variable.

## TEMP

The `TEMP` phaser is a bit of a mirage.  In the [old design documents](https://github.com/Raku/old-design-docs/tree/master?tab=readme-ov-file#perl6-design-documents) it was mentioned in [S06 - Temporization](https://github.com/Raku/old-design-docs/blob/master/S06-routines.pod#temporization).

However it looks like sometime in May 2012 [`let`](https://docs.raku.org/routine/let) and [`temp`](https://docs.raku.org/routine/temp) were implemented.  But the described `TEMP` phaser never was in all of the years since then.  Probably because the functionality of `temp` and `let` covered almost all use cases, combined with the [`KEEP`](https://docs.raku.org/syntax/KEEP) and [`UNDO`](https://docs.raku.org/syntax/UNDO) phasers, so nobody felt the need to actually implement support for it.

Oddly enough the syntax for the `TEMP` phaser *does* exist.
```
$ raku -e 'TEMP say "temp"; say "alive";
alive
```
But it just doesn't do anything.  And after this blog post, chances are that the support for the syntax will be removed (see [problem solvin issue](https://github.com/Raku/problem-solving/issues/511)).

## COMPOSE

The [`COMPOSE` phaser](https://docs.raku.org/syntax/COMPOSE) is even more of a mirage, as it is *not even partly implemented*.  It's only mention is in the [S04 - Phasers](https://github.com/Raku/old-design-docs/blob/master/S04-control.pod#phasers) section of the old design documentation.

However developers wanting to use [roles](https://docs.raku.org/language/glossary#Roles) when composing another [role](https://docs.raku.org/language/objects#index-entry-role_declaration-role) or [class](https://docs.raku.org/language/classtut#Class) often refer to the [lack of its existence](https://irclogs.raku.org/raku/gist.html?2019-12-09Z12:36,2019-12-19Z21:06,2022-08-12Z20:20,2025-05-24Z03:45-0002).

The thing is that most of the functionality of the mythical `COMPOSE` phaser is already available: any code that exists in the mainline of a `role`, will be executed *every time* the role is consumed by a `class` at *compile time*.  A small contrived example:
```raku
role A {
    say "composing $?CLASS.^name()";
}
class B does A { }
class C does A { }
BEGIN say "begin";
```
will show:
```
composing B
composing C
begin
```
Note that the [`$?CLASS`](https://docs.raku.org/language/variables#index-entry-$%3FCLASS) compile time variable contains the type object of the class being composed.  And in August 2020 it basically became clear that a [`COMPOSE` phaser would not be needed](https://irclogs.raku.org/raku/2020-08-12.html#15:28).

## Conclusion

The `DOC` set of phasers is activated with the `--doc` command line argument: without that having been specified, all `DOC` phasers are no-ops but need to be syntactically correct.  Their use appears to be limited.

The `TEMP` phaser was never implemented, but the `temp` and `let` prefix operators provide the functionality promised by that phaser.

The `COMPOSE` phaser was never implemented, because the mainline of a `role` is executed when a `role` is consumed in a class, effectively providing exactly the functionality promised by that phaser.

This concludes the seventh episode of cases of UPPER language elements in the Raku Programming Language.  This also concludes all phasers in Raku.  Next up wil uppercase methods that you, as a user f the Raku Programming Language, can provide in your code to tweak behaviour.  Stay tuned!
