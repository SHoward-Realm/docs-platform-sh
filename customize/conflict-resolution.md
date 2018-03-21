# Conflict Resolution

One of the defining features of mobile is the fact that you can never count on being online. Loss of connectivity is a fact of life, and so are slow networks and choppy connections. But people still expect their apps to work!

This means that you may end up having two or more users making changes to the same piece of data independently, creating conflicts. Note that this can happen even with perfect connectivity as the latency of communicating between the phone and the server may be slow enough that they can end up creating conflicting changes at the same time.

What Realm does in this case is that it merges the changes after the application of specific rules that ensure that both sides always end up converging to the same result even, though they may have applied the changes in different order.

This means that you no longer have the kind of perfect consistency that you could have in a traditional database, what you have now is rather what is termed “strong eventual consistency”. The tradeoff is that you have to be aware of the rules to ensure the consistent result you want, but the upside is that by following a few rules you can have devices working entirely offline and still converging on meaningful results when they meet.

At a very high level the rules are as follows:

* **Deletes always win.** If one side deletes an object it will always stay deleted, even if the other side has made changes to it later on.
* **Last update wins.** If two sides update the same property, the value will end up as the last updated.
* **Inserts in lists are ordered by time.** If two items are inserted at the same position, the item that was inserted first will end up before the other item. This means that if both sides append items to the end of a list they will end up in order of insertion time.

**Primary Keys**

A _primary key_ is a property whose value uniquely identifies an object in a Realm \(just like a primary key in a conventional relational database is a field that uniquely identifies a row in a table\). Primary keys aren’t required by Realm, but you can enable them on any object type. Add a property to a Realm model class that you’d like to use as the primary key \(such as `id`\), and then let Realm know that property is the primary key. The method for doing that is dependent on the language you’re using; in Cocoa, you override the `primaryKey()` class method, whereas Java and .NET use annotations. \(Consult the documentation for your language SDK for more details.\)

Once a Realm model class has a primary key, Realm will ensure that no other object can be added to the Realm with the same key value. You can _update_ the existing object; in fact, you can update only a subset of properties on a specified object without fetching a copy of the object from the Realm. Again, consult the documentation for your language SDK for specifics.

For more information, read the [Primary Keys Tutorial](https://realm.io/news/realm-primary-keys-tutorial/).

**Strings**

{% hint style="info" %}
_Note:_ Strings are not exposed in client APIs yet.
{% endhint %}

Strings are special in that you can see them both as scalar values and as lists of characters. This means that you can set the string to a new string \(replacing the entire string\) or you can _edit_ the string. If multiple users are editing the same string, you want conflicts to be handled at the character level \(similar to the experience you would have in something like Google docs\).

**Counters**

{% hint style="info" %}
_Note:_ Counters are not exposed in all client APIs yet.
{% endhint %}



Using integers for counting is also a special case. The way that most programming languages would implement an increment operation \(like `v += 1`\), is to read the value, increment the result, and then store it back. This will obviously not work if you have multiple parties doing it simultaneously \(they may both read `10`, increment it to `11`, and when it merges you would get the result of `11` rather than the intended `12`\).

To support this common use case we offer a way to express the _intent_ that you are incrementing \(or decrementing\) the value, giving enough hints to the merge that it can reach the correct result. Just as with the strings above, it gives you the choice of updating the entire value, or editing it in a way that conveys more meaning, and allow you to get more precise control of the conflict resolution.

**Custom Conflict Resolution**

The standard way to do custom conflict resolution is to change the value into a list. Then each side can add its updates to the list and apply any conflict resolution rules it wants directly in the data model.

You can use this technique to implement max, min, first write wins, last write wins and pretty much any kind of resolution you can think of.





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

