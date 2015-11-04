<!-- @@@page:manual@@@ -->
<!-- @@@title:Entities@@@ -->

Entities
========

| Module | Namespace | 
|:-------|:----------|
| `npm install giant-entity` | `$entity` |

Giant models the application's data (and parts of its state) in a document-oriented fashion. In this model, Entities are high-level, uniquely identifiable representations of data units of various granularity, having their own APIs for access and manipulation.

Documents, fields, items
------------------------

In Giant, the fundamental entity is the *document*. Documents are semantically atomic representations of real-world entities, such as users, sessions, addresses, campaigns, etc. Documents, and in fact all entities within Giant might have their own internal structure made up of more specific entities. The difference is, that while any entity can be the owner of attribute entities, only documents might have fields.

*Fields*, which represent single properties of documents, might only have a value entity besides its attributes. Based on the field's type, which may be singular (default), or collection, the value might itself have an entity structure of collection items.

*Items*, for convenience reasons, are interchangeable with singular fields, and therefore share the same structure: a value, which in this case cannot contain further items, and other attributes.

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

Let's take a moment to examine the irregular classes: `OrderedCollectionField` and `ReferenceItemKey`.

A normal `CollectionField` implements a general API to access / modify the collection's contents. However, `CollectionField` does not care about the actual contents of the collection, as long as *some* value is associated with each item ID. In an `OrderedCollectionField` however, the values, or the `order` attribute of the values must represent the index of the current item within the ordered collection, and therefore implements a number of extra methods to perform order-related operations.

Collections might be configured in a way that the item ID is set up to be a reference type, ie. the string representation of a `DocumentKey` instance. In these cases, the referenced document key is part of the `ItemKey`, and to avoid unnecessarily repetitive string-to-key conversions, a `referenceKey` property is added to `ReferenceItemKey` instances.

> References are the only way in Giant to 'compose' documents. Sub-documents are an anti-pattern.

### Examples

The following examples creates a keys & entities in various ways. Always use the one that best suits the entities / keys currently available in your application. If you have an entity, and need the key for it, access its `entityKey` property. If you need a key to a child entity, use `getFieldKey` or `getItemKey` methods.

```js
// key to 'user' document of ID '1'
'user/1'.toDocumentKey()

// these resolve to equivalent document instances
'user/1'.toDocument()
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

Entity data is stored in `$entity.entities`, in the semi-structured manner that is expected from a document-oriented database. Documents are grouped by type, and include their fields, and collection items. Note that document types are not collections themselves, as they are in MongoDB for instance. You can't access all documents of a certain type through the entity API, although it is possible to do on a lower level. For this reason, groups of documents must always be referenced from a field on a document that we already have access to.

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

We see 3 documents above, describing 3 fields in the current schema. The documents' type is "field", their IDs are respectively "user/firstName", "user/lastName", and "user/emails".

> When you define your own schema, make sure you append to the config container's "field" node, using `.appendNode()`, to avoid overwriting existing metadata.

Like this:

```js
$entity.config.appendNode('field'.toPath(), {
    "user/organization": { fieldType: "reference" },
    "organization/name": { fieldType: "string" }
});
```

The `reference` type may be assigned to `fieldType`, `itemType`, as well as `itemIdType`. It tells the entity system to treat this field as a reference to another document. 

### Index

What makes an application really *data-driven*, is the ability to query and process entities, and do it fast. To that end, Giant maintains an index container: `$entity.index`. It usually starts out blank, unless there's some index data that is not attainable from the API.

> It's the responsibility of the application to maintain indexes.

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

### Resolving references

Entity events
-------------

Data binding
------------

Custom `Document` classes
-------------------------
