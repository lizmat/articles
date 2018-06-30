Migrating Perl 5 code to Perl 6
===============================

In the coming weeks I will be writing a number of articles about issues you
may encounter if you are a programmer who is programming / has programmed
in Perl 5, when you are doing your first steps in Perl 6.  Most, if not all,
are [already documented](https://docs.perl6.org/language/5to6-nutshell) and
associated documents.  In these articles I will try to go a little more
in-depth about specific issues.

Part 1 - Garbage Collection
===========================
There is *no* timely destruction of objects in Perl 6.  This revelation
usually comes as quite a shock to many people used to the semantics of
object destruction in Perl 5.

Reference counting
------------------

Timely destruction of objects "going out of scope" is achieved in Perl 5
by reference counting.  When something is created in Perl 5, it has at
least a reference count of 1, which keeps it alive.  In its simplest case
it looks like this:

    {
        my $a = 42;  # reference count = 1, because lives in lexical pad
    }
    # lexical pad is gone, reference count to 0

In Perl 5, if the value is an object (aka `bless`ed), it will then get the
`DESTROY` method called on it.

    {
        my $a = Foo->new;
    }
    # $a.DESTROY called

Timely destruction is a very nice feature to have, if you need to deal
with external resources, such as database handles (of which there are
generally only a limited number provided by the database in question).

However, reference counting has several drawbacks.  It has taken Perl 5
*many* years to get all of the reference counting working correctly most
of the time.  And if you're working in `XS`, the reference counting is
something you always need to be aware of to prevent leakage, or to prevent
premature destruction.

Reference counting also has the basic drawback that if two objects contain
references to each other, they will never be destroyed as they keep each
other's reference count above 0 (a circular reference).  In practice, this
often goes much deeper, more like `A -> B -> C -> A`, where A, B and C are
all keeping each other alive.

To circumvent these situations in Perl 5, the concept of *weak reference*
was developed.  Although this can fix the circular reference issue, it
has its performance implications, and doesn't fix the problem of having
circular references in the first place.

Since Perl 6 is a multi-threaded system, it was already decided at a very
early stage that reference counting would be problematic: even in Perl 5
