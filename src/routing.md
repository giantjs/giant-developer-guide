<!-- @@@page:manual@@@ -->
<!-- @@@title:Routing@@@ -->

Routing
=======

| Module | Namespace | Stability |
|:-------|:----------|:----------|
| `npm install giant-routing` | `$routing` | **Stable** |

*Routes* reflect high-level states of the application. They make functional components, like pages, or modules, and their various states identifiable, so that the user may return to find the same component in the same state at a later time by navigating to the same route.
 
In the browser, routes usually manifest in the address bar, which may be controlled directly, or through the browser's navigation controls. (*back*, *forward*, *reload*, *history*, etc.)

![Sample Application Route](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Application%20Route.png)

Giant abstracts route management behind a strongly event-driven mechanism. Based on [universal events](universal-events.md), Giant interprets routes as *paths* on which events may be triggered, signaling changes in the application's current route.

Routes
------

In Giant, routes are represented by instances of the class `$routing.Route`. This class is *evented*, and implements an API for navigation. To create a `Route` instance, use string to `Route` or array to `Route` conversion.
 
```js
'dog/fido/photos'.toRoute()
['dog', 'fido', 'photos'].toRoute()
```

Array to `Route` conversion allows you to instantiate an empty (root) route `[].toRoute()`, which is most useful when capturing routing events triggered on any route.

Navigation
----------

The major role of routes in an application is to provide transparent means of navigating between different states of the application. Giant's routing mechanism allows three different modes of navigation: *immediate*, *asynchronous*, and *debounced*. There's also silent navigation, which immediately alters the route within the application, triggering associated events, but does not update the browser location. This is useful when there is no browser location, e.g. in Node.

Asynchronous navigation is used when we want the application to finish running any synchronous code before proceeding to navigate.

Debounced navigation is designed to prevent routing bursts and loops.

```js
'dog/photos/fido'.toRoute().navigateTo();
'dog/photos/fido'.toRoute().navigateToAsync();
'dog/photos/fido'.toRoute().navigateToDebounced();
'dog/photos/fido'.toRoute().navigateToSilent();
```

Detecting route changes
-----------------------

Giant fires a routing event every time

- the route is about to change, i.e. leaving the current route
- the route *has* changed
- the document finished loading at a specific route

Routing events are instances of `$routing.RoutingEvent`, a subclass of `$event.Event`, and have these two extra properties:

- `beforeRoute`: a `Route` instance representing the route we're navigating away from. On page (re)load, `beforeRoute` is not defined.
- `afterRoute`: a `Route` instance representing the route we're navigating to.

### Leaving route

Events about leaving the current route gives developers the opportunity to prevent the route change from going through.

```js
// prevents navigating to 'dog/photos/fido'
'dog/photos/fido'.toRoute().subscribeTo(
    $routing.EVENT_ROUTE_LEAVE,
    function (event) {
        event.preventDefault();
    });
```

### Route change & document load

Events signaling finished route changes generally manifest as changes in the widget structure: switching pages, or shifting the focus of the current page. In the following example, `$app.PhotosPage` is a subclass of `$basicWidgets.Page`, which we instantiate and display when a relevant route was hit.

```js
'dog/photos'.toRoute().subscribeTo(
    $routing.EVENT_ROUTE_CHANGE,
    function (event) {
        $app.PhotosPage.create().setAsActivePage();
    });
```

Route overrides
---------------

The route change example above assumes that our routes are structured in a certain way (i.e. 'dog/route/:id'). We don't want a routing component to put such constrains on our application's design. In the next example we'll use a different approach, working with route overrides.

Building on Giant's OOP features, it's very easy to create subclasses of `$routing.Route` - representing different domains or pages - and have the correct one instantiate based on the route structure.

If we take our `PhotosPage` example, the subclass would expect the route to start with "dog", "photos" to produce a `PhotosRoute` instance. Once we have our `$app.PhotosRoute` class, extending `$routing.Route`, and optionally adding extra parameters and functionality specific to our "Photos" page, we set up a [surrogate](oop.md#surrogates) definition:

```js
$routing.Route.addSurrogate(
    $app,
    'PhotosRoute',
    function (routePath) {
        var asArray = routePath.asArray;
        return asArray[0] === 'dog' &&
            asArray[1] === 'photos';
    });
```

After this, we can be sure that any route that conforms to this pattern, when converted to a `Route` instance, will get us a `PhotosRoute`.

```js
'dog/photos/fido'.toRoute().isA($app.PhotosRoute) // true
```

We can use this as the basis for determining whether a captured routing event fits our current case.

```js
[].toRoute().subscribeTo(
    $routing.EVENT_ROUTE_CHANGE,
    function (event) {
        if (event.afterRoute.isA($app.PhotosRoute)) {
            $app.PhotosPage.create().setAsCurrentPage();
        }
    });
```

Functionally, this will be equivalent to the one above, but without the unwanted constraints.

PushState vs. hash
-------------------

Giant supports both hash and pushState as the basis for manifesting the route in the URL.

To specify which one is to be used, set the library global flag `usePushState`, *right after* loading the routing module.

```js
$routing.usePushState = true; // or, false
```

[PushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API) looks cleaner in the URL, but requires that possible routes are validated on the static file server serving up `index.html`.

Hash does not require any server-side support, as it relies on the [URL fragment](https://en.wikipedia.org/wiki/Fragment_identifier), but introduces a hash ('#') character between the path to `index.html` and the route string.
