<!-- @@@page:manual@@@ -->
<!-- @@@title:Routing@@@ -->

Routing
=======

| Module | Namespace | 
|:-------|:----------|
| `npm install giant-routing` | `$routing` |

*Routes* are meant to reflect a high-level state of the application. They make functional components, like pages, or modules, and their various states identifiable, so that the user may return to find the same component in the same state at a later time by navigating to the same route.
 
In the browser, routes usually manifest in the URL, which may be controlled directly, or through *back* / *forward* navigation features.

![Sample Application Route](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Application%20Route.png)

Giant abstracts away route management behind a strongly event-driven mechanism. Based on [universal events](universal-events.md), Giant interprets routes as *paths* on which events may be triggered, signaling changes in the application's current route.

Routes
------

In Giant, routes are represented by instances of the class `$routing.Route`. This class is *evented*, and implements an API for navigation. To create a `Route` instance, use string to `Route` or array to `Route` conversion.
 
```js
'dog/fido/photos'.toRoute()
['dog', 'fido', 'photos'].toRoute()
```

Array to `Route` conversion allows you to instantiate an empty (root) route: `[].toRoute()`.

Navigation
----------

The major role of routes in an application is to provide a transparent way for navigating between different states of the application. Giant's routing mechanism allows three different modes of navigation: *immediate*, *asynchronous*, and *debounced*. There's also silent navigation, which immediately alters the route within the application, triggering associated events, but does not update the URL.

Asynchronous navigation is used when we want the application to finish running any synchronous code before proceeding to navigate.

Debounced navigation is designed to prevent routing bursts and loops.

```js
'dog/photos/fido'.toRoute().navigateTo();
'dog/photos/fido'.toRoute().navigateToAsync();
'dog/photos/fido'.toRoute().navigateToDebounced();
'dog/photos/fido'.toRoute().navigateToSilent();
```

Route validation
----------------



Detecting route changes
-----------------------

Giant fires an event every time

- the route is about to change, ie. leaving the current route
- the route *has* changed
- the document finished loading at a specific route

Routing events are instances of `$routing.RoutingEvent`, a subclass of `$event.Event`, and have these two properties:

- `beforeRoute`: a `Route` instance representing the route we're navigating away from
- `afterRoute`: a `Route` instance representing the route we're navigating to

### Leaving route

Events about leaving the current route gives developers the opportunity to prevent the route change go through.

```js
// prevents navigating to 'dog/photos/fido'
'dog/photos/fido'.toRoute()
    .subscribeTo(
        $routing.EVENT_ROUTE_LEAVE,
        function (event) {
            event.preventDefault();
        });
```

### Route change & document load

Events signaling a finished route changes generally manifest as changes in the widget structure, such as switching pages, or changing the focus of the current page. In the following example, `$app.PhotosPage` is an entity-bound widget, which we instantiate and display based on information extracted from the route. 

```js
'dog/photos'.toRoute()
    .subscribeTo(
        $routing.EVENT_ROUTE_CHANGE,
        function (event) {
            var dogId = event.afterRoute
        });
```


Push-state vs. hash
-------------------


