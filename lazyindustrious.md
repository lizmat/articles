Lazy and Industrious Elves
==========================

Christmas is always a busy time of the year for Santa.  Fortunately, Santa has
a lot of helpers.  Always doing little jobs and chores, just to create the
best holiday season experience there is to be!

The [Object::Delayed](https://modules.perl6.org/dist/Object::Delayed) module
adds two more very interesting elves to Santa's merry bunch of elves!  Their
names are `slack` and `catchup`!

Lazy slack
----------

The `slack` elf is very lazy indeed.  It won't do anything until you actually
want to use whatever it was that you asked the `slack` elf to do.  Although
one could consider this a very bad character trait in an elf, it's also a
very ecological trait.  One could consider the `slack` elf to be the greenest
elf of them all!  How many times have you asked an elf to do something for you
and then not used the result of the hard work of that elf?  Even though it's
only recycled electrons that are being moved around, it still costs energy to
move them around!  Especially if those electrons are used to tell other elves
to do something far away, like in an external database!

    use Object::Delayed;
    my $dbh = slack { DBIish.connect(...) }

That's what you need to have a `$dbh` variable that will only make a connection
to a database when it is actually needed.  Of course, if you want to make a
query to that database, that can also be made to slack!

    use Object::Delayed;
    my $dbh = slack { DBIish.connect(...) }
    my $sth = slack { $dbh.prepare(...) }

Since the statement handle is also slacked, it won't actually do the query
preparation until actually needed.

    use Object::Delayed;
    my $dbh = slack { DBIish.connect(...) }
    my $sth = slack { $dbh.prepare(...) }
    # lotsa program
    if $needed {
        $sth.execute;  # opens database handle + prepares query
    }

So if `$needed` is true, calling the `.execute` function will make the `$sth`
become a real statemement handle after having made `$dbh` a real database
handle.  Isn't that great?  Because if you didn't need it, all the elves doing
the query preparation could be doing other things, and the elves making the
database connection could **also** be doing other things.  Not to mention the
elves of the database being blissfully ignorant of your initial plan to make
a database connnection at all!

Of course, if you **did** need the database connection, it is always a good
idea to tell the elves of the database that you're done.  In Perl 6, this
doesn't happen automatically, because Santa doesn't keep track of how much
each elf is doing.  Santa likes to delegate responsibility!  You typically
tell the database elves that you're done when you leave the section of code
in which you needed the database handle.

    LEAVE .disconnect with $dbh;

The `LEAVE` elf is special in that it will do the stuff it is told to do when
you leave the block in which `LEAVE` elf is called.  In this case the
`.disconnect` method is called on `$_` if the `$dbh` is defined: the `with`
elf not only tests if the given value is defined, bit **also** sets `$_`.

But, but, but, won't checking whether `$dbh` make the connection to the
database?  No, the `slack` elf is smart enough that if you're asking if
something if `.defined`, or `True` or `False`, it will **not** actually
start doing the work for you.  Which is **sooo** different from the `catchup`
elf!


