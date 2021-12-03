Santa's OCD sorted
------------------

Santa has been around for a long time already.  Santa remembers the days when bits where set by using a magnetic screwdriver.  In those days, you'd made sure that things were orderly set up and sorted for quick access.

Santa likes the Raku Programming Language a lot, because it just works like Santa thinks.  There's just this one thing missing to make Santa feel at home again, just like in the olden days: an easy way to make sorted lists and easily insert new values into these lists to keep them up-to-date.

Sure, Santa knows there are hashes.  And if you want to iterate over all keys alphabetically sorted, you can easily do:
````raku
for %hash.keys.sort -> $key {
    ...
}
````
or if you want both the key and the value:
````raku
for %hash.sort(*.key) -> (:$key, :$value) {
    ...
}
````
But that just feels like a lot of extra work on big hashes that were filled organically from keys and associated values.

So Santa went looking in the [Raku ecosystem](https://raku.land) and was really glad when the [Array::Sorted::Util](https://raku.land/zef:lizmat/Array::Sorted::Util#name) distribution propped up on the search term ["sort"](https://raku.land/?q=sort).

So what does that do?  Well, it exports a few subroutines, the most simple one is [inserts](https://raku.land/zef:lizmat/Array::Sorted::Util#inserts).  You give it an array and an object, and it will insert it in the array at the correct location to keep the array sorted:
````raku
use Array::Sorted::Util;

my str @names;
inserts(@names,$_) for <Zaphod Arthur Ford>;
say @names; # [Arthur Ford Zaphod]
````
But what if you would have a list of `Pair`s with names and gifts?  You'd need **two** arrays, and they would have to be kept in sync!  Well, that is also easily be possible with `inserts` Santa found out:
````raku
use Array::Sorted::Util;

my str @names;
my str @gifts;
for Zaphod => 'arm', Arthur => 'tea', Ford => 'blanket' {
    inserts(@names, .key, @gifts, .value);
}
say @names;  # [Arthur Ford Zaphod]
say @gifts;  # [tea blanket arm]
````
And then, if you want to look up the gift of a specific person, you'd use [finds](https://raku.land/zef:lizmat/Array::Sorted::Util#finds):
````raku
say @gifts[$_] with finds(@names, 'Arthur');  # tea
say @gifts[$_] with finds(@names, 'Marvin');  # (no output)
````
So Santa made changes to the code to use two lists instead of a hash.  But the elves **really** didn't like that.  So they went searching in the Raku ecosystem as well, and found [Array::Sorted::Map](https://raku.land/zef:lizmat/Array::Sorted::Map#name).  This allowed them to easily use a `Map` interface to the two lists:
````raku
use Array::Sorted::Map;
my %gifts := Array::Sorted::Map.new(keys => @names, values => @gifts);
%gifts = Zaphod => 'arm', Arthur => 'tea', Ford => 'blanket';
.say with %gifts<Arthur>;  # tea
.say with %gifts<Marvin>;  # (no output)
````
That was good as a temporary measure.  But it still wouldn't allow the elves to make changes to the hash without having to resort to things like [finds](https://raku.land/zef:lizmat/Array::Sorted::Util#finds), [inserts](https://raku.land/zef:lizmat/Array::Sorted::Util#inserts) or [deletes](https://raku.land/zef:lizmat/Array::Sorted::Util#deletes) operating on the underlying arrays.

The wise Santa realized that the elves have the future, so it was important to find a way that elves as well as Santa would be happy with.  A further search revealed the existence of a [Hash::Sorted](https://raku.land/zef:lizmat/Hash::Sorted#name) module, which promised to keep the keys of a hash created with that class, to always be in sorted order.

When Santa proposed to have the elves use that module, they were very glad.  Now to could use the familiar hash idioms **and** satisfy Santa's need for order:
````raku
use Hash::Sorted;

my %gifts is Hash::Sorted;
my @names := %hash.keys;
my @gifts := %hash.values;

%gifts = Zaphod => 'arm', Arthur => 'tea', Ford => 'blanket';
say @names;  # (Arthur Ford Zaphod)
say @gifts;  # (tea blanket arm)

.say with %gifts<Arthur>;  # tea
.say with %gifts<Marvin>;  # (no output)
````
What the elves didn't realize, was that the [Hash::Sorted](https://raku.land/zef:lizmat/Hash::Sorted#name) module is just a frontend for the subroutines provided by [Array::Sorted::Util](https://raku.land/zef:lizmat/Array::Sorted::Util#name).  But Santa wisely didn't tell that to the elves.

And all was well on the North Pole!
