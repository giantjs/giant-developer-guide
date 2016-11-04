<!-- @@@page:manual@@@ -->
<!-- @@@title:Entities@@@ -->

Entities
========

| Module | Namespace | Stability |
|:-------|:----------|:----------|
| `npm install giant-entity` | `$entity` | **Stable** |

Giant models the application's data (and parts of its state) in a document-oriented fashion. In this model, *entities* are high-level, uniquely identifiable representations of data units of various granularity, having their own APIs for access and manipulation.

Documents, fields, items
------------------------

In Giant, the fundamental entity is the *document*. Documents are semantically atomic representations of real-world entities, such as users, sessions, addresses, campaigns, etc. Entities possess internal structure, made up of *attributes*. Certain entity types have special attributes: documents have *fields*, fields have *value*, *collections* have *items*, and so on.

> By default special attributes are identical to the parent entity.

Documents' special attribute is the *fields* entity. A flat, associative list of *field* entities. Besides *fields*, the document might have other attributes. By default, the document entity is identical to its *fields* attribute, and has no other attributes.

The field entity's special attribute is the *value*, which holds the actual value associated with the field. For singular fields, the value is either a primitive, or an object with non-repetitive structure. For collection fields, the value is an *items* entity, holding a flat, associative list of *item* entities. By default, the field entity is identical to its *value* attribute, and has no other attribute.

Item entities behave the same way as field entities, except items can't be other than singular. They have attributes, including *value*, to which they're identical much like field entities. For convenience reasons, items are interchangeable with fields on the code level: they share the same base class and API. 

![Document Structure](https://raw.githubusercontent.com/giantjs/giant-developer-guide/master/images/Document%20Structure.png)

> Each entity is identified by an associated *key*.

The key contains sufficient information to identify an entity.
 
- for documents: a *document type* and *document ID*
- for fields, the parent document's key and a *field name*
- for items, the parent field's document key, field name, and it's own *item ID*

The document ID is expected to uniquely identify a document within its document type. A field name is expected to identify a field within its parent document, and similarly, an item ID is expected to identify a collection item relative to its parent collection field.

A specific Giant class implements each of the above. `$entity.EntityKey` and `$entity.Entity` are the base classes for keys and entities, respectively.

- for documents: `DocumentKey`, and `Document`
- for fields: `FieldKey`, `Field`, `CollectionField`, and `OrderedCollectionField`
- for items: `ItemKey`, `ReferenceItemKey`, and `Item`
- for general attributes: `Attribute`, and `AttributeKey` 

~~Check the API documentation for implementation details.~~

Let's take a moment to examine the irregular classes: `OrderedCollectionField` and `ReferenceItemKey`.

A normal `CollectionField` implements a general API to access / modify the collection's contents. However, `CollectionField` does not care about the actual contents of the collection, as long as *some* value is associated with each item ID. In an `OrderedCollectionField` however, the values, or the `order` property of the value nodes must represent the index of the current item within the ordered collection, and therefore implements a number of extra methods to perform order-related operations.

Collections might be configured in a way that the item ID is set up to be a reference type, ie. the string representation of a `DocumentKey` instance. In these cases, the referenced document key is part of the `ItemKey`, and to avoid unnecessarily repetitive string-to-key conversions, a `referenceKey` property is added to `ReferenceItemKey` instances.

> References are the only way in Giant to 'compose' documents. Sub-documents are not supported.

### Examples

The following examples create keys and entities in various ways. Always use the one that best suits the entities or keys currently available in your application. If you have an entity, and need the key for it, access its `entityKey` property. If you need a key to a child entity, use `getFieldKey` or `getItemKey` methods.

```js
// key to 'user' document of ID '1'
'user/1'.toDocumentKey()
['user', '1'].toDocumentKey()

// these resolve to equivalent document instances
'user/1'.toDocument()
['user', '1'].toDocument()
'user/1'.toDocumentKey().toDocument() 
'user/1'.toDocumentKey().toEntity()

// these resolve to equivalent field keys
'user/1/firstName'.toFieldKey()
'user/1'.toDocumentKey().getFieldKey('firstName')
'user/1/firstName'.toField().entityKey
'user/1'.toDocument().getField('name').entityKey

// gets key to home email of user/1
'user/1/emails/home'.toItemKey() // item key

// gets key to 'emails' field of user/1 from item key
'user/1/emails/home'.toItemKey().getFieldKey()
```

Entity storage
--------------

Data associated with entities resides in a central datastore, composed of three containers, each being an instance of `$data.Tree`.

### Entities

Entity data is stored in `$entity.entities`, in the semi-structured manner that is expected from a document-oriented database. Documents are grouped by type, and include their fields, and collection items. Note that document types are not collections themselves, as they are in MongoDB for instance. You can't access all documents of a certain type through the entity API, although it is possible to do on a lower level.

For this reason:

> Groups of documents must always be referenced from a field on a document that we already have access to.

The JSON below illustrates the contents of the `entities` container.

```json
{
    "user": {
        "1": {
            "firstName": "John",
            "lastname": "Smith",
            "emails": {
                "home": "john.smith@homeemail.com",
                "office": "john.smith@officeemail.com"
            },
            "organization": "organization/1"
        }
    },
    "organization": {
        "1": {
            "name": "Smithcorp, Inc."
        }
    }
}
```

### Metadata

Descriptive information *about* documents, fields, and items are kept in `$entity.config`, in the same document structure as `$entity.entities`. Currently it's mainly reserved for specifying types of fields, item IDs, and item values. Giant's entity system uses these settings to decide how to process fields.

```json
{
    "field": {
        "user/firstName": { "fieldType": "string" },
        "user/lastName": { "fieldType": "string" },
        "user/emails": {
            "fieldType": "collection",
            "itemType": "string"
        }
    }
}
```

We see three documents above, describing three fields in the current schema. The documents' type is "field", their IDs are "user/firstName", "user/lastName", and "user/emails", respectively.

> When you define your own schema, make sure you append to the config container's "field" node, using `.appendNode()`, to avoid overwriting existing metadata.

Like this:

```js
$entity.config.appendNode('field'.toPath(), {
    "user/organization": { fieldType: "reference" },
    "organization/name": { fieldType: "string" }
});
```

The `reference` type may be assigned to `fieldType`, `itemType`, and `itemIdType`. It tells the entity system to treat this field as a reference to another document. The `reference` type is reserved for forward compatibility. 

### Index

What makes an application really *data-driven*, is the ability to query and process entities, and do it fast. To that end, Giant maintains an index container: `$entity.index`. It usually starts out blank, unless there's some index data that is not attainable through the API.

> It's the application's responsibility to maintain indexes.

A typical start-of-word search index would look like this:

```js
{
    "full-text": {
        "j": {
            "o": {
                "h": {
                    "n": {
                        "references": {
                            "user/1": "user/1"
                        }
                    }
                }
            }
        }
    }
}
```

Using this index and a simple `Query` we can find documents associated with indexed strings that start with "jo":

```js
var query = 'full-text>j>o>\\>references>|'.toQuery();
$entity.index.queryValues(query);
// ["user/1"]
```

### Management

The `config` container requires little management. Usually, it is set up once, based on static, hard-coded data, and remains that way throughout the application's entire life cycle.

Managing The `entities` container is a bit more complex, as there's a good chance we want to reset its contents occasionally, either to clean up sensitive, user-related data on logout, or just to conserve memory.

> When resetting any of the containers, we must make sure to restore the initial content.

The entities container is updated either by a merge process, which integrates (after possibly transforming) API responses into the application state, or, directly via entity API, depending on whether the application implements ~~lazy API calls~~.

The most complex management is required by `$entity.index`. Not only does it need to be reset from time to time, but its contents require constant attention to remain in sync with the entities being indexed.

Usually, an index is managed by a single class. This class must take care of 

1. **initialization**, by setting up the initial structure on first access,
2. **event subscription**, to get notifications of relevant entity changes,
3. **synchronisation**, by modifying index data according to entity changes carried by subscribed events

In maintaining an index the data flow is always uni-directional. Modifying an index does not trigger any events.

Entity access & manipulation
----------------------------

Now that we can create keys and entity instances, and know where entities are being stored, it's time to connect the two concepts.

Among the many responsibilities of a key instance is to resolve the information that it stores and identifies an entity, to a specific path in the `entities` container. For this purpose, every key class implements a `getEntityPath` method, returning a `$data.Path` instance. The mapping between keys and entity paths must be [bijective](https://en.wikipedia.org/wiki/Bijection), however, paths are never actually resolved to keys, because paths lack structure and hence the additional information that would be required to tell what kind of entity it represents.

```js
'user/1/firstName'.toFieldKey().getEntityPath().toString()
// document>user>1>firstName
```

Through the above example key strings and path string look very similar, however this would change with the introduction of entity attributes.

Entity instances can get and set entity nodes relying on the path information obtained from the associated key (`.entityKey`). To get the entity node, one would have to call `.getNode()` on it, to set it, `.setNode()` respectively. So far the entity API seems to resemble that of `$data.Tree`, except here we're not passing any path information. The same similarity may be observed for appending nodes, and node removal, as `.appendNode()`, `.unsetNode()`, and `.unsetKey()` are also implemented on `Entity`.

### Fields and items

While `.setNode()` is the standard way of setting entity data on all entity classes, `Field` (and its subclass `Item`) implements the `.setValue()` shorthand to set the node right on its *value* attribute. (Which, as discussed above, by default, is equivalent to the field or item itself.)

> For forward compatibility reasons it's always safer to use `Field.setValue()` and `Item.setValue()` to set the values of fields and items.

The expression `'user/1/firstName'.toField().setNode("Robert")` might have a very different effect than `'user/1/firstName'.toField().setValue("Robert")` if the field has other attributes.

### Resolving references

A central problem when working with entities, especially if the data is coming from a REST API, is resolving references.

> Having a key to an entity does not mean having the entity data, too.

Resolving a key to actual data usually leads through three steps, managed by a different component, eg. Giant's ~~API Access~~ layer.
 
1. Resolving the key to an API resource or endpoint
2. Invoking the endpoint and fetching response
3. Merging response into entity container

Only when these three steps have succeeded can we attempt again to access the data associated with our key.

> When resolving references, the application must always be prepared to handle the no-data case.

Entity events
-------------

Entity manipulation triggers events in the event space `$entity.entityEventSpace`. Attempting to access an absent node via the entity API triggers `$entity.EVENT_ENTITY_ACCESS`, changing a node triggers `$entity.EVENT_ENTITY_CHANGE`. All events are triggered on the entity path prepended with 'entity', serving as root path for all entity events.

The following example logs all entity changes, including the affected key, the before, and after values. 

```js
$entity.entityEventSpace
    .subscribeTo(
        'entity'.toPath(), // subscribing at root
        $entity.EVENT_ENTITY_CHANGE, // to changes
        function (event) {
            console.log(
                "entity changed:",
                 entity.sender.toString(), 
                 event.beforeNode,
                 event.afterNode);
        });
```

The event's `sender` for entity events holds the associated entity key. Event instance properties `beforeNode` and `afterNode` are specific to entity change events, implemented by the `$entity.EntityChangeEvent` class. 

With the above event subscription in place, here's what we'd get by updating a single field:

```js
'user/2/firstName'.toField().setValue("Jen");
// entity changed: user/2/firstName undefined Jen
```

Setting the same value again (based on strict equality) would not trigger another event.

Entity key classes are evented, which means they offer simplified means for subscribing to entity events.

```js
'user/2/firstName'.toFieldKey()
    .subscribeTo(
        $entity.EVENT_ENTITY_CHANGE,
        function () {
            console.log("entity changed");
        });
```

Access events are often used for kicking off on-demand fetching from the back end API. In certain cases, eg. when testing the presence of an entity, access events are undesirable. Class `Entity` implements the method `getSilentNode`, which works exactly like `.getNode()`, except for triggering access events.

Data binding
------------

Binding manages event subscriptions for class instances. It maintains a registry of event handlers relevant to the *bound* instance, indexed by event names, method names, and in the case of entities, entity keys. Binding offers considerable flexibility over the regular event subscription API, in that it allows handler functions to be wrapped, potentially adding extra functionality on top of the specified handler methods.

In the context of entities, binding endows application components to react to entity changes. There are three different kinds of data binding in Giant:
 
- **Simple replacement binding**, triggering the handler only when the observed entity node gets replaced: `.bindToEntityChange()`,
- **Content binding**, capturing any change within the observed node: `.bindToEntityContentChange()`,
- And **delegated binding**, where changes are captured on nodes being closer to the root of the observed path, but are treated as if they occurred right there: `.bindToDelegatedEntityChange()`.

> Data binding in Giant is agnostic about how entity data gets updated, and has nothing to do with API access.

Classes must have the `$entity.EntityBound` trait in order to be data-bound. This trait provides the binding API, and maintains the bindings.

The following example illustrates delegated binding. The first update only changes Fido's 'hungry' field from `false` to `true`. The second update overwrites the entire Fido document with the property 'hungry' set to `false`.

```js
var HungryDog = $oop.Base.extend()
    .addTrait($entity.EntityBound)
    .addMethods({
        init: function (dogKey) {
            $entity.EntityBound.init.call(this);
            this.bindToDelegatedEntityChange(
                dogKey.getFieldKey('hungry'), 
                'onHungryChange');
        },
        
        onHungryChange: function (event) {
            var dogKey = event.sender,
                isHungry = event.afterNode,
                dogId = dogKey.documentId;
            if (isHungry) {
                console.log("feed", dogId, "!");
            } else {
                console.log(dogId, "is kinda full");
            }
        }
    });
    
var fidoKey = 'dog/fido'.toDocumentKey(),
    hungryDog = HungryDog.create(fidoKey);

'dog/fido/hungry'.toField().setValue(true);
// feed fido !

'dog/fido'.toDocument().setNode({ hungry: false });
// fido is kinda full

hungryDog.unbindAll();
```

It is important to unbind once the instance is no longer in use. With components that have a life cycle, binding and unbinding usually takes place in life cycle callbacks.

Custom `Document` classes
-------------------------

For convenience reasons, applications are encouraged to implement `Document` overrides, in order to provide domain-specific APIs for documents.

Imagine in our user example, that we can call specific, semantically named functions to fetch or modify entity data.

```js
'user/1'.toDocument().getFirstName()
// "John"

'user/1'.toDocument().getHomeEmail() 
// "john.smith@homeemail.com"
```

To do this, first we need to define the `Document` override.

```js
var UserDocument = $entity.Document.extend()
    .addMethods({
        getFirstName: function () {
            return this.getField('firstName')
                .getValue();
        },
        getHomeEmail: function () {
            return this.getField('emails')
                .getItem('home')
                .getValue();
        }
    });
```

Once we have that, the next and final step would be to direct any 'user' document instantiation to our override class. After this, the expressions above will work as expected.

```js
$entity.Document
    .addSurrogate(
        window,
        'UserDocument',
        function (documentKey) {
            return documentKey.documentId === 'user';
        });
```
