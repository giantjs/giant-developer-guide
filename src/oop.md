<!-- @@@page:manual@@@ -->
<!-- @@@title:OOP@@@ -->

OOP
===

The module [`giant-oop`](https://github.com/giantjs/giant-oop) implements Giant's class system with utilities. Associated namespace: `$oop`.

Giant is a strongly object oriented framework. The OO paradigm permeates all of its components: with very few exceptions, everything is a *class*, *trait*, or *interface*. Giant makes use the *prototypal* nature of JavaScript to build classes, however, it goes around the language's built-in *classical*, single inheritance pattern mostly associated with constructor functions and the `new` keyword. Giant introduces its own class system based on [prototypal inheritance](https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain), which comes with powerful concepts and tools assisting the way classes are built, instantiated, and tested. These concepts include (quasi-) multiple inheritance, surrogates, memoization, reflection, and built-in mocks.

*ES6 classes*, which bring virtually no extra functionality, only syntax on top of their arguably inflexible ES5 counterparts, are considered an anti-pattern within Giant.

Visibility
----------

Giant's OOP layer recognizes two levels of visibility: *public*, and *private*. This distinction however, is descriptive. There are no [*actual* privates](http://javascript.crockford.com/private.html) in Giant, mainly for performance reasons. Instead, 'privates' are non-enumerable, prefixed properties on the class or instance they're applied to.

> Since privates are practically accessible to anything with access to the class, a number of rules must be observed to avoid unexpected behavior.

- Only the class *introducing* a private member should use that private member. Other classes, including the subclasses of the private member's host class, must not call them.
- Subclasses should not override private members.
- Private members should be named as uniquely as possible to avoid collision with private members in base / subclasses.

Most modern IDEs and code quality tools will point out problems related to privates.

`$oop.Base`
-----------

`Base` is the base class of all classes throughout Giant modules, as well as the modules of the application. It introduces a set of basic methods with which any class, much like a crane, builds itself. Methods that have to do with adding properties / methods are grouped by visibility: `.addMethods()`, `.addPrivateConstants()`, etc.
 
~~Check the API reference for specific methods.~~

Single inheritance
------------------

Single inheritance, in a prototypal model, does nothing more than add a blank prototype level (through `Object.create()`) to host properties that will define the extended class.

Giant implements single inheritance through the `.extend()` method.

![Single Inheritance](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Single%20Inheritance.png)

### Example

    var base = $oop.Base.extend(),
        host = base.extend();

Multiple inheritance with traits
--------------------------------

Since the prototypal inheritance is constrained by the chain structure, and multiple inheritance is not supported by the language itself (not even ES6), a workaround is required to achieve anything that resembles it.

Giant facilitates multiple inheritance by introducing *traits*.

> Traits are self-contained, shallow classes that lend their methods and properties to the host class.

Giant implements trait addition through `.addTrait(trait)` and `.addTraitAndExtend(trait)` methods. While the former merely mixes the properties of the specified trait to the host (except for `.init`), the latter calls an `.extend()` afterwards, as traits might implement the same methods as the base or the host, and extending is the only way to avoid collision.

![Multiple Inheritance](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Multiple%20Inheritance.png)

### Example

    var base = $oop.Base.extend(),
        trait = $oop.Base.extend(),
        host = base.extend()
            .addTrait(trait);

Adding properties & methods
---------------------------

The base class `$oop.Base` implements the methods for classes to build their APIs, by adding properties and methods of different levels of visibility. ~~Check API for specific property addition methods.~~

Each non-static class must have an `.init()` method, which receives the constructor arguments.

It is very important not to override class-building methods.

### Example

In this example we're adding a static string property (`foo`), and a public method (`baz`). 
    
    var MyClass = $oop.Base.extend()
        .addConstants({
            foo: "bar"
        })
        .addMethods({
            baz: function (quux) {}
        });        

Overriding methods
------------------

Methods in the host class that override those of the base, must call their super methods explicitly.

Methods in the host class that override those of the base, must call their super methods **as well as** the trait's implementation (if any).
        
Instantiation
-------------

Surrogates
----------

Priority

Memoization
-----------

Memoization with surrogates
---------------------------

Reflection
----------

Built-in mocks
--------------

Utilities
---------

### Postponed definitions

### Extending built-ins

### Adding module-global properties

Configuration
-------------

$oop.privatePrefix

$oop.messy

$oop.testing