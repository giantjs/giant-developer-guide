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

Fields, which represent single properties of documents, might only have a value entity besides its attributes. Based on the field's type, which may be singular (default), or collection, the value might itself have an entity structure of collection items.

Items, for convenience reasons, are interchangeable with singular fields, and therefore share the same structure: a value, which in this case cannot contain further items, and other attributes.

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

Entity access & manipulation
----------------------------

### Resolving references

The entity store
----------------

Entity events
-------------

Data binding
------------

Custom `Document` classes
-------------------------
