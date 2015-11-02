<!-- @@@page:manual@@@ -->
<!-- @@@title:Data Structures@@@ -->

Data Structures
===============

| Module | Namespace |
|:--------|:----------|
| `npm install giant-data` | `$data` |

Higher-level modules in Giant do considerable data transformations mostly for maintaining lookups, registries, and indexes. The basic tools and structures that perform such transformations are implemented in `giant-data`.

`$data.Hash`
------------

The base class for any component that stores and processes complex data is `$data.Hash`. The purpose of `Hash` is serve as a common denominator for converting more specific data structures based on it to each other. Through `Hash` we're able to convert a `Collection` into a `Dictionary`, `Dictionary` to `Tree`, and so on, as long as they share the same base class, `Hash`.

Internally, `Hash` maintains the buffer for the data being operated on, (`items`, which is either an `Object` or `Array`) and implements very generic methods for manipulating the hash, including clearing, cloning, low-level data access, etc.

When introducing new data types in the application domain, it makes sense choose `Hash` as their basis to ensure smooth interoperability.

### Conversion methods

`Hash`, as the lowest common denominator for all classes representing structured data, is also the host for conversion methods between them. Conversion methods take the buffer (`items`) and wrap it in a different - `Hash`-based - API. This way, APIs can remain specific to the data they're working with, but still compatible, allowing flexibility to treat data differently at different points of the transformation process.

In the following example we apply a dictionary transformation (reverse), convert the result to a `Collection` using `Hash.toCollection()` in order to iterate over and output the reversed key-value pairs.

```js
$data.StringDictionary.create({
    dog: ['Fido', 'Buddy', 'Rover'],
    cat: 'Fluffy'
})
.reverse()
.toCollection()
.forEachItem(function (name, species) {
    console.log(name, species);
});

// will output:
// Fido dog
// Buddy dog
// Rover dog
// Fluffy cat
```

Higher level structures
-----------------------

Giant implements a number of `Hash`-based structures in order to cover most transformations. ~~Check API documentation for details.~~

- **Collection, specified collection**: Implements item manipulation & extraction, iteration, mapping, and filtering over a hash buffer. Buffer values equate to item values. So-called "specified collections" allow to fuse the `Collection` API with the API of a specified type, such as `Array`, `Date`, and `String`.
- **Dictionary, string dictionary**: Implements item manipulation & extraction, where item values equate to buffer values when they are not arrays, or buffer value items when they are.
- **Ordered list, ordered string list**: Implements item & range manipulation, and basic search for ordered lists of any type, and strings specifically.
- **Set**: Implements basic set operations: union, intersection, subtraction, difference.
- **Chain**: Implements chain structure, where inserting / removing items is cheap but locating them is expensive.
- **Tree**: Implements tree manipulation and querying for matching keys, values, paths, and their combinations. More about trees below.

Filtering collections
---------------------

Filtering is one of the most frequent operations performed on collections. Filtering iterates over the collection's items and returns a new `Collection` instance (actually an instance of the class of the original collection) with key-value pairs from the original collection satisfying the given predicate.
 
Giant implements different filtering methods on `Collection` for different filtering bases.

Mapping collections
-------------------

Creating specified collections
------------------------------

Combining dictionaries
----------------------

Reversing a dictionary
----------------------

Manipulating trees
------------------

The most powerful among fundamental data structures is the tree. It allows the user to manipulate and query deep object structures. Trees are generally used for temporary storage throughout the application to implement datastores, lookups, and indexes. Among others, it is the basis for the evented [~~Entity framework~~](entities.md). 

Circular nodes are not supported.

We'll use the following data in our examples, our meeting schedule for next week.

```js
var schedule = $data.Tree.create({
    "Monday": {
        "9:00": "huddle",
        "10:00": "John",
        "15:00": "Sarah"
    },
    "Tuesday": {
        "8:45": "huddle",
        "11:00": "CEO"
    },
    "Wednesday": {
        "9:00": "huddle"
    },
    "Thursday": {
        "9:00": "huddle",
        "9:30": "planning",
        "16:30": "Sarah"
    },
    "Friday": {
        "9:00": "huddle",
        "12:00": "Jen"
    }
});
```

In a `Tree`, each node is uniquely identified with a path, composed of the keys associated with all its parent nodes all the way up to the root. For instance, The value `"CEO"` in our example could be accessed as `schedule.items.Tuesday["11:00"]`. The part with "Tuesday.11:00" is the path relative to the root of the tree. While accessing nodes like this works when the data is there, it fails with an exception thrown when we try to access a node that's not there, eg. `schedule.items.Saturday["9:00"]`. To this end, Giant introduces its own way of accessing nodes and managing paths, one which does not fail in similar circumstances.

Paths in Giant are instances of the `Path` class, which provides methods for manipulating the path. The path above would look like this:

```js
'Tuesday>11:00'.toPath()
```

Keys within the path's string representation are separated by a greater-than sign (`>`).

This path instance can be then used to access the node, which will return "CEO".

```js
schedule.getNode('Tuesday>11:00'.toPath()); // "CEO"
```

And respectively, a path where there is no node, returns `undefined`.

```js
schedule.getNode('Saturday>9:00'.toPath()); // undefined
```

Paths may also be used in addressing a node for change. For instance, the following expression changes "CEO" to "CTO" in our `schedule` tree.

```js
schedule.getNode('Tuesday>11:00'.toPath(), "CTO");
```

Querying trees
--------------

Getting & setting single nodes is usually enough to manipulate a tree, but to work with the data inside, we need to traverse the tree and collect nodes that satisfy a given set of conditions. In Giant, those conditions are expressed by queries.

The `Query` is a subclass of `Path`, sharing a common structure, but instead of an exact match for each key, a query can define patterns that match multiple nodes.

These patterns are:

- `'|'`: Wildcard, matching all strings for a singe key.
- `'<'`: Option separator, when specifying multiple options for exact key match.
- `'"'`: Leaf node, matches keys with primitive values.
- `'^'`: Key-value separator, for exact value matches. Allows any key pattern (exact, options, wildcard) to be used.
- `'\\'`: Skips keys on the path until the next pattern is matched. Not greedy, ie. matches the current key if there's no pattern coming after.
- `'{'`, `'}'`: Marks the key to be included in the result set.

Applied to the sample data above, here are a few examples:
 
```js
// matches 11:00 meetings for all days
'|>11:00'.toQuery()

// matches daily schedule for Monday and Friday
'Monday<Friday'.toQuery()

// matches the root node
'\\'.toQuery()

// matches all leaf nodes in the tree
'\\>"'.toQuery()

// matches all nodes with "huddle" as value
'\\>|^huddle'.toQuery()

// matches all schedule nodes that have "Sarah" in them
'{|}>|^Sarah'.toQuery()
```

`Tree` implements quite a few methods for querying, in order to distinguish between different types of keys and values returned.

Querying:

- keys using `.queryKeys()` simply returns all matched keys in an array, discarding all information about the exact location of the matched keys in the tree.
- values using `.queryValues()` simply returns all matched values in an array, discarding all information about the exact location of the matched keys in the tree.
- paths using `.queryPaths()` returns an array of path instances for all paths matched.
- key-value pairs via `.queryKeyValuePairs()` returns an object, with the matched values, indexed by their respective keys. The returned object can have no more than one occurrence of each key, so if the query matched multiple key occurrences, some of those will be overwritten.
- path-value pairs using `.queryPathValuePairs()` returns an object, with the matched values, indexed by the string representation of their respective paths. This will return all matched nodes separately.

Each query method has its `Hash` version, in which the result is wrapped in a `Hash` instance. (`.queryValuesAsHash()`, `.queryKeysAsHash()`, etc.) Use this when you need to go on processing the results as different data structures.

In the example below, we want to know when the huddle starts each day. 

```js
schedule.queryPathsAsHash('|>|^huddle'.toQuery())
    .toCollection()
    .mapKeys(function (path) {
        return path.asArray[0];
    })
    .mapValues(function (path) {
        return path.asArray[1];
    })
    .items;
```

Which will return:

```json
{
    "Monday": "9:00",
    "Tuesday": "8:45",
    "Wednesday": "9:00",
    "Thursday": "9:00",
    "Friday": "9:00"
}
```

So we'll know not to be late for the huddle on Tuesday.
