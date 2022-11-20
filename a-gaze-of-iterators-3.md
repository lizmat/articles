# A gaze of iterators! (Part 3)

This is the third part of the ["A gaze of iterators!"](https://dev.to/lizmat/a-gaze-of-iterators-4pg5) series.

## By any other name

Let's look at the names of the iterators in part 2.  For this, I'm going to use the [`.^name`](https://docs.raku.org/routine/name#class_Metamodel::DefiniteHOW) method.  As we've seen before, the `.^foo` means "calling the `.foo` method on the object's meta-object".
```
say <a b c>.iterator.^name;
# Rakudo::Iterator::ReifiedListIterator
say (1..*).iterator.^name;
# Rakudo::Iterator::IntRangeUnending
say (1..10).pick(*).iterator.^name;
# List::PickN
say (1..10).map({++$_}).iterator.^name;
# Any::IterateOneWithoutPhasers
```
As you can see, the names of the classes of these iterator objects are all over the place.  But they all provide the same interface: being able to call methods such as `.pull-one`, `.push-all` and `.sink-all`.

In many programming languages, you'd expect all of these classes to be sharing the same parent classes.  In the Raku Programming Language you can indeed inherit from a parent class.  The way to check which parents a class has, is with the [`.^mro`](https://docs.raku.org/routine/mro) method (for **m**ethod **r**esolution **o**rder).
```
say <a b c>.iterator.^mro;
# ((ReifiedListIterator) (Any) (Mu))
say (1..*).iterator.^mro;
# ((IntRangeUnending) (Any) (Mu))
say (1..10).pick(*).iterator.^mro;
# ((PickN) (Any) (Mu))
say (1..10).map({++$_}).iterator.^mro;
# ((IterateOneWithoutPhasers) (Any) (Mu))
```
That is odd?  They all seem to inherit from [`Any`](https://docs.raku.org/type/Any) and [`Mu`](https://docs.raku.org/type/Mu)?  Yet, one can **not** call the `.pull-one` method on every object that just inherits from `Any` and `Mu`:
```
say 42.^mro;
# ((Int) (Cool) (Any) (Mu))
say 42.pull-one;
# No such method 'pull-one' for invocant of type 'Int'
```

## Role playing

The Raku Programming Language also provides a thing called ["roles"](https://docs.raku.org/language/objects#Roles).  In short, you could think of a role as a collection of methods that will be "implanted" into a class if the class itself does not provide a method implementation for it.  All of these iterator classes that we've seen here, actually do the [`Iterator`](https://docs.raku.org/type/Iterator) role.  And just as with the `^.mro`, you can introspect which roles a class performs by calling the `.^roles` method.  Let's see how that works out here:
```
say <a b c>.iterator.^roles;
# ((PredictiveIterator) (Iterator))
say (1..*).iterator.^roles;
# ((Iterator))
say (1..10).pick(*).iterator.^roles;
# ((Iterator))
say (1..10).map({++$_}).iterator.^roles;
# ((SlippyIterator) (Iterator))
```
So it looks like some classes are actually playing more than one role.  But they *all* also do the `Iterator` role, it looks like.

## How to be an iterator

To make a `class` be an iterator, one must tell the class to do the `Iterator` role.  That's pretty simple, no?  Let's start with an empty class that just wants to be an iterator.  You do that by using [`does`](https://docs.raku.org/language/objects#index-entry-does):
```
class Foo does Iterator {
}
===SORRY!=== Error while compiling -e
Method 'pull-one' must be implemented by Foo
because it is required by roles: Iterator.
```
So we need to actually provide some type of implementation for the interface that the `Iterator` role is providing.  Ok, so let's make a very simple method `pull-one` that will randomly return `True` or `False`:
```
class TrueFalse does Iterator {
    method pull-one() { Bool.roll }
}
say TrueFalse.pull-one;  # True | False
```
The `.roll` method randomly picks a single value from a set of values.  When called on an [`enum`](https://docs.raku.org/language/typesystem#index-entry-Enumeration-_Enums-_enum), it will randomly [select one of the enums values](https://docs.raku.org/routine/roll#role_Enumeration).  And the `Bool` enum has `True` and `False` as its values.

Of course, this is all very boring, let's make it more interesting:
```
class YeahButNoBut does Iterator {
    method pull-one() {
        Bool.roll ?? "Yeah but" !! "No but"
    }
}
say YeahButNoBut.pull-one;  # Yeah but | No but
```
So we now have a class that produces an iterator.  But how would you actually use that in any "normal" way in your program?  Well, by embedding the iterator into another class, and have a method `.iterator` in it that returns the iterator class:
```
class Jabbering {
    method iterator() {
        my class YeahButNoBut does Iterator {
            method pull-one() {
                Bool.roll ?? "Yeah but" !! "No but"
            }
        }
    }
}
```
Note here that the `.iterator` method actually returns the class.  Because that's all we need from this iterator class.

Also note that classes in the Raku Programming Language can be lexically scoped by prefixing them with `my`, just as you would lexically scoped variables.  This makes sense in this case, as there would be no need for the iterator class outside of the scope of the "Jabbering" class.

So now you can start jabbering!
```
.say for Jabbering;
```
Hmmm... that doesn't stop now, does it?

Indeed it doesn't.  As to why, that's for the next instalment in this series!

## Conclusion

This concludes the third part of the series, in which the concept of roles in the Raku Programming Language is introduced, along with `does`.  And that you can alter the scope of a `class` by prefixing it with `my`.

Questions and comments are always welcome.  You can also drop into the [#raku-beginner channel](https://web.libera.chat/?channel=#raku-beginner) on Libera.chat, or on Discord if you'd like to have more immediate feedback.

I hope you liked it!  Thank you for reading all the way to the end.
