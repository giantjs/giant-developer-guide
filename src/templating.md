<!-- @@@page:manual@@@ -->
<!-- @@@title:Templating@@@ -->

Templating
==========

| Module | Namespace | Stability |
|:-------|:----------|:----------|
| `npm install giant-templating` | `$templating` | **Stable** |

In the context of Giant, templates are strings; where the content of marked parameters is deferred to a resolution process. Giant implements its own templating engine, built on common principles with other Giant modules, such as the concept of `Stringifiable`'s and [universal events](universal-events.md).

It's important to note that the primary purpose of Giant's templating engine is *not* markup generation. Giant, as a framework, unlike [Angular](https://angularjs.org/) or [Ember](http://emberjs.com/), is not template-based. Giant expects all application logic to be implemented with code (JavaScript), eliminating the need for any templating 'language', beyond the usual handlebars-style parameter markers. (Other modules in Giant provide templating support specifically for HTML markup, but those are also quite different from common templating engines.)

Static templates
----------------

Static templates hold template strings and facilitate their resolution - based on supplied parameters. They're implemented by the `$templating.Template` class.

Here's a typical use of a static template:

```js
'Hello {{what}}!'.toTemplate().getResolvedString({
    '{{what}}': "World"
}) // "Hello World!"
```
Parameter values can be either:

1. Strings
2. Types that may be converted to string ad-hoc (number, boolean)
3. Objects with meaningful `.toString()` override
4. Classes that implement `$utils.Stringifiable`. (This is essentially the same as 3, but in a way that conforms to Giant's OOP guidelines.)

Typically, apart from actual strings, you'd pass instances of `$templating.LiveTemplate` (see below), [`$i18n.Translatable`](i18n.md), and [`$entity.Field`](entities.md#fields-and-items) or `$entity.Item` as parameter values.

`Template` itself implements `Stringifiable`, returning the template string when converted.

```js
'Hello {{what}}!'.toTemplate() + "" // "Hello {{what}}!"
```

### Parameter nesting

Substitutions might become more troublesome as the complexity of our strings increase. One typical problem, especially in the field of localization, is translating expressions that are mixed with HTML. Say, we want the substitution "World" to appear emphasized in our final string, but for localization purposes, we don't want to concern ourselves with the HTML tags involved.

We'll solve this problem by introducing an additional parameter, nested into the original one.

```js
'Hello {{what}}!'.toTemplate().getResolvedString({
    '{{what}}': '<em>{{plain-what}}</em>',
    '{{plain-what}}': "World"
}) // 'Hello <em>World</em>!'
```

Then, when we send strings to the translation specialist, we'll send `"Hello {{what}}"`, and `"World"`, but not `"<em>{{plain-what}}</em>"`.

> Circular nesting will cause stack overflow, and must be avoided.

Live templates
--------------

In reality, static templates are so basic that they're rarely used in application code. They serve, however, as the foundation for 'live' templates, which, in turn:

- carry substitution parameters with them
- convert to the resolved template string when stringified
- trigger events on parameter change

Live templates are implemented by `$templating.LiveTemplate`, also a `Stringifiable`.

```js
'Hello {{what}}!'.toLiveTemplate().setParameterValues({
    '{{what}}': "World"
}) + "" // "Hello World!"
```

On the surface, this example seems to be doing exactly the same as our first static example above with `.getResolvedString()`. The output *is* the same, but the mechanism is different. Here we're getting the resolved template by *object to string* conversion, instead of ad-hoc substitution.

> A live template can be re-used, its parameters changed, and re-resolved multiple times.

### Parameter nesting

Parameter nesting works the same way for live templates as for static. When we set the parameter values on the live template, parameter names may appear in values for other parameters.

There is a recurring practice concerning live templates and nesting: it's a good idea to apply nesting parameters on initialization, and change variable parameters on demand.

Again, the use case is localization, but it goes the same way for any other purpose.

```js
var template = 'Hello {{what}}!'.toLiveTemplate()
    .setParameterValues({
        '{{what}}': '<em>{{plain-what}}</em>'
    });

// ...
template.setParameterValues({
    '{{plain-what}}': "World"
});
template + "" // Hello <em>World</em>!

// ...
template.setParameterValues({
    '{{plain-what}}': "GiantJS"
});
template + "" // Hello <em>GiantJS</em>!
```

### Detecting parameter changes

The most powerful feature of live templates is that they notify whenever parameter values change. The class `LiveTemplate` bears the trait `$event.Evented`, and thus offers the usual API (shared by entity keys, routes, etc.) for event subscriptions. Through these events, dependants such as [widgets](widgets.md) displaying template-based strings on the UI get a chance to update themselves.

Using the `template` instance from the previous example,

```js
template.subscribeTo(
    $templating.EVENT_TEMPLATE_PARAMETER_VALUES_CHANGE,
    function (event) {
        console.log("params changed, new string: " +
            event.sender);
    });

template.setParameterValues({
    '{{plain-what}}': "World"
});
// > "params changed, new string: Hello <em>World</em>!"

template.setParameterValues({
    '{{plain-what}}': "GiantJS"
});
// > "params changed, new string: Hello <em>GiantJS</em>!"
```

First, we subscribed to parameter changes in our `template` instance, then we changed a parameter two times, triggering the subscribed handler on each occasion. As `event.sender` in the handler is a reference to our `LiveTemplate` instance `template`, the string conversion will get us the resolved string.

In order for each template to trigger events individually, `LiveTemplate` also has the `Documented` trait, assigning unique IDs to each instance. These instance IDs are then used in the path on which parameter change events are triggered. The root of the all template paths is 'template', and the event space in which they're triggered is the shared `$event.eventSpace`. If necessary, we may capture all template parameter changes by subscribing in the event space directly. (Notice how we subscribe *before* any template instance is created.)

```js
$event.eventSpace.subscribeTo(
    $templating.EVENT_TEMPLATE_PARAMETER_VALUES_CHANGE,
    'template'.toPath(),
    function (event) {
        console.log("params changed, new string: " +
            event.sender);
    });

'Good {{what}}'.toLiveTemplate()
    .setParameterValues({
        '{{what}}': "morning"
    });

// > "params changed, new string: Good morning"
```
