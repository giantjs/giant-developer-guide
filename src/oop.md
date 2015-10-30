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

Giant implements single inheritance through the `extend` method.

![Single Inheritance](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Single%20Inheritance.png)

### Example

```js
var base = $oop.Base.extend(),
    host = base.extend();
```

Multiple inheritance with traits
--------------------------------

Since the prototypal inheritance is constrained by the chain structure, and multiple inheritance is not supported by the language itself (not even ES6), a workaround is required to achieve anything that resembles it.

Giant facilitates multiple inheritance by introducing *traits*.

> Traits are self-contained, shallow classes that lend their methods and properties to the host class.

Giant implements trait addition through `.addTrait(trait)` and `.addTraitAndExtend(trait)` methods. While the former merely mixes the properties of the specified trait to the host (except for `.init`), the latter calls an `.extend()` afterwards, as traits might implement the same methods as the base or the host, and extending is the only way to avoid collision.

![Multiple Inheritance](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Multiple%20Inheritance.png)

### Example

```js
var base = $oop.Base.extend(),
    trait = $oop.Base.extend(),
    host = base.extend()
        .addTrait(trait);
```

Adding properties & methods
---------------------------

The base class `$oop.Base` implements the methods for classes to build their APIs, by adding properties and methods of different levels of visibility. ~~Check API for specific property addition methods.~~

> Each non-static class must have an `init` method, which is called on instantiation.
 
The `init` method receives the arguments passed to the constructor and initializes the instance.

> It is very important not to override class-building methods, such as `.extend()`, `.create()`, `.addMethods()`, etc.

### Example

In this example we create a `Dog` class, endowing it with constant 4 legs, and the ability to bark. 
    
```js
var Dog = $oop.Base.extend()
    .addConstants({
        legCount: 4
    })
    .addMethods({
        bark: function () {
            alert("Woof!");
        }
    });        
```

Overriding methods
------------------

When subclasses override methods of their respective base classes or traits that they're composed of, the host's implementation must invoke all relevant implementations explicitly in order to get their combined functionality. Invoking base and trait implementation must pass the current instance as context, via `Function.prototype.call` or `Function.prototype.apply`, as well as any relevant arguments.

> Failure to call base / trait implementations may lead to unexpected behavior.
        
### Example

Here we implement the `Dog` class as an extension of `Animal`, that has the ability to move. Then, we override the dog's movement method by adding panting after it moved.

```js
var Animal = $oop.Base.extend()
    .addMethods({
        move: function (distance) { }
    });        

var Dog = Animal.extend()
    .addMethods({
        pant: function () {},
        move: function (distance) {
            Animal.move.call(this, distance);
            this.pant();
        }
    });
```
        
Instantiation
-------------

Classes are instantiated by calling their `create` method, passing any constructor arguments.

### Example

The `Dog`'s `init` method receives the dog's name and sets it as an instance property.

```js
var Dog = $oop.Base.extend()
    .addMethods({
        init: function (name) {
            this.name = name;
        }
    });
    
var fido = Dog.create("Fido");
fido.name // "Fido"
```

Constructor memoization
-----------------------

The framework's assumption about the application is, that it will be composed of many classes and even higher number of class instances. Instantiation however, might have performance implications as constructor logic gets more and more complex.

> To that end, Giant offers a way to balance *performance* against *memory footprint* by caching instances.

By telling a class to maintain a registry of instances, indexed by strings based on constructor arguments, we're able to fetch an existing instance instead of creating a new one. To set this up, we only need to set an *instance mapper* function, which will generate the key that identifies the instance in the registry.

For example, the `InsuredPerson` class implemented below, will create only one instance for each SSN passed to the constructor. Each subsequent call to `InsuredPerson.create()` with the same SSN will yield the instance already associated with that SSN.

```js
var InsuredPerson = $oop.Base.create()
    .setInstanceMapper(function (ssn) {
        return ssn;
    });    
    
var pete = InsuredPerson.create(1),
    jen = InsuredPerson.create(2),
    who = InsuredPerson.create(1);
```

The following statements will be true:

```js
pete !== jen
who === pete
who !== jen    
```

> It is imperative that the instance IDs generated by the mapper function are actually identifying each instance, otherwise we might get unintended instances, or even completely inhibit the creation of certain instances.

This also implies a practice that is followed throughout Giant's code, and should be followed throughout the application as well:

> Constructor arguments should identify the instance. Anything else should be set via setter methods.

It might seem convenient to pass the initial state to an instance through the constructor, but this is strongly advised to refrain from.

### Singleton

A special case of memoization is the singleton pattern. In order to ensure that a class has but one instance, use the instance mapper function below.

```js
var Singleton = $oop.Base.create()
    .setInstanceMapper(function () {
        return "singleton";
});
```

Following the guidelines above, singleton classes should zero constructor arguments.

### Clearing instance cache

As the application goes on operating for a long time without full-page reload, instance registries might become quite sizable, increasing memory footprint above desirable levels.

One *preventive* practice to deal with this is to memoize only those classes that might have a finite number of possible instances, so their memory footprint is capped. Singletons belong in this category, the SSN example above does not.

The alternative to this is to *clear* the registry (registries) at specific points of the application's lifecycle. For a single class, this is as simple as (using our sample classes from above): `InsuredPerson.clearInstanceRegistry()`, or `Singleton.clearInstanceRegistry()`. For clearing all memoized classes, which is a good practice to do on logout for instance, first we need to collect all such classes, and call `clearInstanceRegistry` on each. ~~Check "Best practices" on how to do this.~~

Surrogates
----------

One of the most powerful features of Giant is its ability to forward instantiation of a base class to one of its subclasses, based on constructor arguments.

Say, we have a base class, `Dog`, that takes the breed as well as the name as constructor arguments. In this setup, `Dog` has a subclass: `Chihuahua`, with slightly different implementation, but we don't know up front what breed our dog will be.

```js
var Dog = $oop.Base.extend()
        .addMethods({
            init: function (name, breed) {
                this.name = name;
                this.breed = breed;
            },
            bark: function () {
                alert("Woof!");
            }
        }),
    Chihuahua = Dog.extend()
        .addMethods({
            bark: function () {
                alert("Eeek!");
            }
        });
```

Now, we could get the correct class with a [factory pattern](http://www.oodesign.com/factory-pattern.html) like this:

```js
function getDog(name, breed) {
    switch (breed) {
    case 'chihuahua':
        return Chihuahua.create(name, breed);
    default:
        return Dog.create(name, breed);
    }
}

getDog('Buddy', 'beagle').bark(); // "Woof!"
getDog('Fido', 'chihuahua').bark(); // "Eeek!"
```

What Giant's surrogate mechanism does instead, is that it takes *filter functions* based on constructor arguments, runs them in a certain order on instantiation of the *base* class, then ends up creating an instance of first class where the filter function returns `true`. 

> The great advantage of surrogates over factories is that they can be set up in a distributed way.

In our case, we only need to set up a surrogate between `Dog` and `Chihuahua` to make things work.

```js
Dog.addSurrogate(window, 'Chihuahua',
    function (name, breed) {
        return breed === 'chihuahua';
    });

Dog.create('Buddy', 'beagle').bark(); // "Woof!"
Dog.create('Fido', 'chihuahua').bark(); // "Eeek!"
```

### Surrogate priority

The order in which surrogate filter functions are evaluated follow their addition, which is usually not fixed. To provide a way to control the order, Giant introduces the *priority* attribute associated with each surrogate filter. To set up surrogates with priority, an extra numeric argument will be passed to `.addSurrogate()`, which defaults to 0. Higher priority surrogates will be evaluated first.

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