<!-- @@@page:manual@@@ -->
<!-- @@@title:Universal Events@@@ -->

Universal Events
================

| Module | Namespace | Stability |
|:-------|:----------|:----------|
| `npm install giant-event` | `$event` | **Stable** |

In a system of interacting components, inversion of control, and separation of concerns are cornerstones of scalability and clarity of design. A mechanism through which self-contained components listen to shared structures to which other components may communicate changes in their state, implements both features. In Giant, these shared structures are called *event spaces*, traversed by *events* along *paths*, in a process called *bubbling*.

The purpose of Giant's event module is to generalize eventing to any application component or data structure that can be expressed through or mapped to a path. Drawing on analogy with the DOM, parent-children relationships between nodes define such a path.

Other examples are:

- application *routes*, such as `'users/john-smith'`, or `'articles/weather/04042015'`
- API resources, such as `'session/1234abcd'`
- internal datastore keys, such as `'user/1/fullName'`
- file paths such as `'images/logo.png'`

Unlike DOM events, paths and events in Giant are not tied to the structures they represent. An event may be triggered on a path even when there is nothing corresponding to it. This feature comes handy when the purpose of an event is to signal the absence of something, which would not be possible in the DOM for instance.

![Event Bubbling](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Event%20Bubbling.png)

Terms
-----

- **Event**: Signals a change in the state of a component. Events, as opposed to *messages*, which are not covered here, do not have specific recipients. Events really are meant to express things that just happened, without concern as to who gets it.
- **Event space**: Shared structure traversed by events.
- **Sender**: Any component of the application communicating changes in its state to an event space.
- **Listener**: Any component of the application listening to senders' events.
- **Source**: Specific component on which events may be triggered.
- **Target**: Specific component on which events may be captured.
- **Event path**: Identifies a component of the application in the context of an event space.
- **Bubbling**: The process of events traversing an event space. Handlers are invoked at each stage of the path. Bubbling is optional but is switched on by default.
- **Evented**: A class or instance of a class that has the `$event.Evented` trait.

Event space
-----------

The event space is the structure in which events are triggered and listened to. Each event is created within the confines of an event space, and cannot cross between event spaces. The event space governs the event's traversal within the space. Event spaces are usually specific to the feature the events of which they transmit: routing, back end API, datastores, etc.

Giant introduces a general purpose event space instance: `$event.eventSpace`. Unless we expect substantial number of subscriptions to be associated with a specific application component, such as entities or widgets, this basic event space will be sufficient for triggering events in. Otherwise we'll need a new `EventSpace` instance to be created for such components, eg. `$entity.entityEventSpace`, or `$widget.widgetEventSpace` for entities and widgets, respectively.

> It's a good idea to have the first (root) key in an event path identify the component, eg. 'entity' in `'entity>document>user>1>fullName'`, to make sure event paths are unique within a shared event space.

Event
-----

`Event` is the central concept and class of Giant's event mechanism. Each event is endowed with an *event name* and is permanently associated with an event space. Events may be created ad-hoc, as: `$event.Event.create()`, but this is in fact rarely used. Events are, by and large, spawned by a structure implementing the `EventSpawner` interface, like event spaces and evented classes.

Spawning an event spares us the burden of:

1. having to associate the event with an event space, as well as
2. preparing the event with special properties, payload, etc.

### Example

```js
var event = $event.eventSpace.spawnEvent();
```

Triggering events
-----------------

To trigger an event, first one must be created or spawned, then have any extra information set via either its API or as a payload, and then call `.triggerSync()` on it. The example below illustrates this process (including spawning) on an event space instance.

```js
$event.eventSpace
    .spawnEvent('dog.bark')
    .triggerSync('dog>Fido'.toPath());
```

Or, on an instance of an `Evented` class: (More about `Evented` classes later on.)

```js
fido
    .spawnEvent('dog.bark')
    .triggerSync();
```

As the examples show, events are triggered synchronously. This means that all handlers associated with this event will be called before execution proceeds to the line after the trigger. In the second example no path argument is being passed to `.triggerSync()`, as the evented instance already has that information.

Implementing asynchronous triggering is up to the application, by using `setTimeout`, or promises, etc.

Broadcasting events
-------------------

Events normally bubble from the end of the affected path toward its root. In certain situations though, we might want to notify a number of subscribed components at once. Giant makes this possible, as long as the components in question share a common root on their event paths.

Here's how we'd notify the application that an entire pack of dogs is barking, assuming that the event path associated with a single dog follows the `'dog>packName>dogName'` structure, and there events are being listened to in `$event.eventSpace`:

```js
$event.eventSpace
    .spawnEvent('dog.bark')
    .broadcastSync('dog>101dalmatians'.toPath());
```

> Broadcasting an event invokes all corresponding handlers subscribed on paths *relative* to the target path.

Handlers will be called in an undetermined order, after which the event will bubble from the target path as if it was triggered there.

> Use broadcasting when the intended listener is not known at the time of triggering te event.

**Use broadcasting sparingly.** Depending on the number and complexity of subscribed handlers, the process might have considerable performance implications, especially, when done frequently. Combine broadcasting with techniques like de-bouncing, and keep handlers few and light.

Subscribing to events
---------------------

There are multiple ways to subscribe to events. The following code samples lead to the same result: logging the message "route changed from home to user" when the application route changes from 'home' to 'user'. ~~See the Routing chapter for more information on routing events.~~

**Directly on an event space**

```js
function onRouteChange (event) {
    console.log(
        "route changed from",
        event.beforeRoute.toString(),
        "to",
        event.afterRoute.toString());
}

$routing.routingEventSpace.subscribeTo(
    $routing.EVENT_ROUTE_CHANGE, // event name
    'user'.toPath(),             // capture path
    onRouteChange);              // handler
```

**On an `Evented` class or instance**

The `$routing.Route` class has the `Evented` trait, and may be instantiated by conversion from an array or string. Notice that in this case we don't need to specify the event path, as it is carried by the `Evented` (`Route`) instance itself.

```js
'user'.toRoute().subscribeTo(
    $routing.EVENT_ROUTE_CHANGE, // event name
    onRouteChange);              // handler
```

Overriding `Event`
------------------

It's very rare that events are not subclassed, as it helps

1. to identify events by purpose, and 
2. extend the event's API with domain-specific methods and properties.
 
For instance, the ~~Routing~~ module's `RoutingEvent` adds route-specific properties and methods that make processing the event in a handler much easier.

When subclassing `Event`, it's usually a good idea to set up surrogates based on either event name, or event space, or both. This would make sure that events spawned with specific names or on specific event spaces end up being the right type. The default implementation(s) of `EventSpawner.spawnEvent()` instantiate the base event class. (One may also override `spawnEvent()`, but that's and arguably less robust solution.)

A suitable surrogate might look look this:

```js
var DogEvent = $event.Event.extend();

$event.Event.addSurrogate(
    window,
    'DogEvent', 
    function (eventName, eventSpace) {
        return eventName && 
            eventName.substr(0, 3) === 'dog';
    });
    
$event.eventSpace.spawnEvent('dog').isA(DogEvent)
// true

$event.eventSpace.spawnEvent('cat').isA(DogEvent)
// false
```

Evented classes
---------------

The trait `$event.Evented` was brought into existence to formalize and simplify implementation of classes the instances of which are to be represented by a path in a specific event space. A good example for this are ~~widgets~~. Each widget has a path associated with it, made up of the unique identifiers of their parent widgets. Events may be triggered on evented instances without having to specify the event space or event path, because both of those are already defined and tied to the *evented* instance.

The following example illustrates:

- assigning the `Evented` trait to a class
- setting the event space and event path on an instance
- subscribing to a specific event on the evented instance
- triggering an event on the evented instance

```js
var Dog = $oop.Base.extend()
        .addTrait($event.Evented);
    
Dog.addMethods({
    init: function (name) {
        this.setEventSpace($event.eventSpace)
            .setEventPath(['dog', name].toPath());
    }
});

var fido = Dog.create('Fido');

fido.subscribeTo('dog.bark', function (event) {
    var dogName = event.originalPath.getLastKey();
    console.log(dogName, "barked");
});

fido.triggerSync('dog.bark');
// "Fido barked"
```

Note that we subscribe to Fido's events *statically*, ie. outside of the instance's life cycle. This is to make sure we're not subscribing more than once. In case our instance does have a life cycle, as is the case with widgets for example, subscribing at the start of the cycle and unsubscribing at the end is a good practice.

The importance of method elevation
----------------------------------

Event subscription methods take a handler function as one of their arguments, a function that will end up being called back by the internal event mechanism. In an object oriented setup, one is usually passing methods to event subscriptions, but as we know in JavaScript doing so makes the method lose its context. Unless, of course, it's bound to one. Ad-hoc binding is simple, but inconvenient, especially if you need the re-use the function reference at unsubscription. Giant's [OOP](oop.md) module implements method elevation - binding methods to the instance in a reusable way - which is heavily encouraged in evented and listener classes.

The example below re-implements the dog-barking scenario from before with methods for handlers.

```js
var Dog = $oop.Base.extend()
        .addTrait($event.Evented);
    
Dog.addMethods({
    init: function (name) {
        this.elevateMethod('onBark');
    
        this.setEventSpace($event.eventSpace)
            .setEventPath(['dog', name].toPath());
    },
    
    startListening: function () {
        this.subscribeTo('dog.bark', this.onBark);
        return this;
    },
    
    stopListening: function () {
        this.unsubscribeFrom('dog.bark', this.onBark);
        return this;
    },
    
    onBark: function (event) {
        var dogName = event.originalPath.getLastKey();
        console.log(dogName, "barked");
    }
});

var fido = Dog.create('Fido');
    
fido.startListening();

fido.triggerSync('dog.bark');
// "Fido barked"

fido.stopListening();

fido.triggerSync('dog.bark');
// no response
```

Tracing events
--------------

Very often the handling of an event depends on the causal chain that led to triggering that event. To this end, Giant implements an `originalEvent` property on event instances. This way, different action may be taken when reacting to events, eg. discerning whether a route change was caused by user navigation, or landing on the route for the first time.

The distinction is made by looking at the chain of original events, and discerning the relevant origin. For instance, looking at the last original event:

```js
// ...
onBark: function (event) {
    var originalEvent = event.originalEvent;
    switch (originalEvent.eventName) {
    case 'dog.hungry':
        console.log("barking b/c hungry");
        break;
    // ...
    }
}
// ...
```

or, reaching down in the chain of original events to see if there was a specific event leading to triggering the current one:

```js
// ...
onBark: function (event) {
    var originalEvent = event
        .getOriginalEventByName('sky.thunder');
    if (originalEvent) {
        console.log("barking b/c scared");
    }
}
// ...
```

In order to have a meaningful original event on our 'barking' handler, our class must subscribe to the appropriate event, eg. `sky.thunder`, and trigger a bark event in response.

If the secondary event is triggered synchronously, there's nothing special to be done in the handler, other than (eventually) triggering the event.

```js
// ...
onThunder: function () {
    this.triggerSync('dog.bark');
}
// ...
```

However, when the secondary event is fired asynchronously, the handler must return a [*thenable*](http://wiki.commonjs.org/wiki/Promises/A), which gets resolved / rejected *after* the secondary event is triggered.

The example below uses a [Q](https://github.com/kriskowal/q) promise to do this.

```js
// ...
onThunder: function (event) {
    var deferred = Q.defer();
    // this is how long it takes for Fido to get scared
    setTimeout(function () {        
        this.triggerSync('dog.bark');
        deferred.resolve();       
    }, 500); 
    return deferred.promise;
}
// ...
```

Event payload
-------------

`Event` classes usually have the means of carrying relevant extra information through instance properties. ~~Routing~~ events implement `.beforeRoute` and `.afterRoute`, entity events add `.beforeNode` and `.afterNode`. In certain cases though, the application needs to hang some additional information on an event in an ad-hoc fashion. Because of this, all events have a `payload` property, which can be added to either on the event, or through a global payload store.

Adding payload to an ad-hoc event:

```js
var fido = Dog.create('Fido');
fido.spawnEvent('dog.bark')
    .setPayloadItem('loudness', '100dB')
    .triggerSync();
```

Adding payload to an event that will be triggered eventually, by name:

```js
var fido = Dog.create('Fido');
$event.setNextPayloadItem('dog.bark', 'loudness', '100dB');
fido.spawnEvent('dog.bark').triggerSync();
$event.deleteNextPayloadItem('dog.bark', 'loudness');
```

> When multiple sources set or remove the same payload item on the same event name, it might get removed prematurely. Make sure that the same payload item is not set by different sources.
