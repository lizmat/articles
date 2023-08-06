# Garbage Collection
In this blog post we’ll get into how garbage collection in Raku differs from Perl.

## No timely destruction
There is **no** timely destruction of objects in Raku. This revelation usually comes as quite a shock to people used to the semantics of object destruction in Perl. But worry not, there are other ways in Raku to get the same behaviour, albeit requiring a little more thought by the developer. Let’s first examine a little background on the situation in Perl.

## Reference counting
In Perl, timely destruction of objects "going out of scope" is achieved by [reference counting](https://en.wikipedia.org/wiki/Reference_counting). When something is created in Perl, it has a reference count of 1 or more, which keeps it alive. In its simplest case it looks like this:
```
# Perl
{
    my $a = 42;  # reference count of $a is equal to 1,
                 # because lives in lexical pad
}
#  lexical pad is gone, reference count to 0
```
In Perl, if the value is an object (aka blessed), the `DESTROY` method will be called on it as soon as the reference count reaches 0:
```
# Perl
{
    my $a = Foo->new;
}
# $a->DESTROY called
```
If no external resources are involved, timely destruction is just another way of managing memory used by a program. And you, as a programmer, shouldn’t need to care about how and when things get recycled. Having said that, timely destruction is a very nice feature to have if you need to deal with external resources, such as database handles (of which there are generally only a limited number provided by the database server). And reference counting can provide that.

However, reference counting has several drawbacks. It has taken Perl core developers many years to get reference counting working correctly. And if you’re working in XS, you always need to be aware of reference counting to prevent memory leakage or premature destruction.

Keeping things in sync gets more difficult in a multi-threaded environment, as you do not want to lose any updates to references made from multiple threads at the same time (as that would cause memory leakage and/or external resources to not be released). To circumvent that, some kind of locking or atomic updates would be needed, neither of which are cheap.

Please note that Perl ithreads are more like an in-memory fork with unshared memory between interpreters than threads in programming languages such as C. So, it still doesn’t need any locking for its reference counting.

Reference counting also has the basic drawback that if two objects contain references to each other, they will never be destroyed as they keep each other’s reference count above 0 (a circular reference). In practice, this often goes much deeper, more like A -> B -> C -> A, where A, B, and C are all keeping each other alive.

The concept of a weak reference was developed to circumvent these situations in Perl. Although this can fix the circular reference issue, it has performance implications and doesn’t fix the problem of having (and finding) circular references in the first place. You need to be able to find out where a weak reference can be used in the best way; otherwise, you might get unwanted premature object destruction.

## Reachability analysis
Since Raku is multi-threaded in its core, it was decided at a very early stage that reference counting would be problematic performance-wise and maintenance-wise. Instead, objects are evicted from memory when more memory is needed and the object can be safely removed.

In Raku you can create a `DESTROY` method in a class, just as you can create a `DESTROY` sub in a package that is used for a blessed object in Perl. But in Raku you can **not** be sure when it will be called (if ever).

Without getting into too much detail, objects in Raku are destroyed only when a garbage collection run is initiated, e.g., when a certain memory limit has been reached. Only then, if an object cannot be reached anymore by other objects in memory and it has a `DESTROY` method, will it be called just prior to the object being removed.

No garbage collection is done by Raku when a program exits. Applicable phasers (such as `LEAVE` and `END`) **will** get called, but no garbage collection will be done other than what is (indirectly) initiated by the code run in the phasers.

If you always need an orderly shutdown of external resources used by your program (such as database handles), you can use a phaser to make sure the external resource is freed in a proper and timely manner.

For example, you can use the `END` phaser (known as an `END` block in Perl) to disconnect properly from a database when the program exits (for whatever reason):
```
# Raku
my $dbh = DBIish.connect(…) or die "Couldn’t connect";
END dbh.disconnect;
```
Note that the `END` phaser does not need to have a block (like { ... }) in Raku. If it doesn’t, the code in the phaser shares the lexical pad (lexpad) with the surrounding code.

There is one flaw in the code above: if the program exits before the database connection is made or if the database connection failed for whatever reason, it will still attempt to call the `disconnect` method on whatever is in $dbh, which will result in an execution error.

There is however a simple idiom to circumvent this situation in Raku using [`with`](https://docs.raku.org/language/control#with_orwith_without).
```
# Raku
END .disconnect with $dbh;
```
The postfix `with` matches only if the given value is defined (generally, an instantiated object) and then topicalises it to `$_`. The `.disconnect` is short for `$_.disconnect`.

If you would like to have an external resource clean up whenever a specific scope is exited, you can use the `LEAVE` phaser inside that scope.
```
# Raku
if DBIish.connect(…) -> $dbh {
    # store handle in $dbh
    LEAVE $dbh.disconnect;
    # do your stuff with the database
}
else {
    say “Could not do stuff that needed to be done”;
}   
```
Whenever the scope of the `if` is left, any `LEAVE` phaser in that scope will be executed. Thus the database resource will be freed whenever the code has run in that scope.  Even if it caused an execution error!

## Summary
Even though Raku does not have the timely destruction of objects that Perl users are used to, it does have easy-to-use alternative ways to ensure management of external resources, similar to those in Perl.
