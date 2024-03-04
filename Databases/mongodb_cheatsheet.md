# MONGODB CHEATSHEET

1. CONNECT VIA MONGOSH
2. MONGODB DATABASES AND COLLECTIONS (show, use, create, drop, stats, rename)
3. MONGODB CRUD (insert, find, update, delete)
4. MONGODB OPERATORS (query comparison, field update, read modifiers)
5. MONGODB INDEXES (get, create, drop, hide, unhide)
6. MONGODB USER MANAGEMENT (show, auth, get, create, drop, update, grant, remove, revoke)

## 1. CONNECT VIA MONGOSH

<details><summary>Connecting to a Mongo database via mongosh cli.</summary>

```sh
mongosh # connects to mongodb://127.0.0.1:27017 by default
```

```sh
mongosh --host <host> --port <port> --authenticationDatabase admin -u <user> -p <pwd> # omit the password if you want a prompt
```

```sh
mongosh "mongodb://<user>:<password>@192.168.1.1:27017"
```

```sh
mongosh "mongodb://192.168.1.1:27017"
```

```sh
mongosh "mongodb+srv://cluster-name.abcde.mongodb.net/<dbname>" --apiVersion 1 --username <username> # MongoDB Atlas
```

</details>

## 2. MONGODB DATABASES AND COLLECTIONS

<details><summary>Show and use databases and collections.</summary>

Show databases.

```sh
show dbs
```

Switch to a database.

```sh
use <database_name>
```

Show collections.

```sh
show collections
```

</details>

<details><summary>Create and drop databases and collections.</summary>

To create a new database, you can use the `use` command and a non-existing name.

```sh
use <database_name>
```

Delete the current database. Double check that you are *NOT* on the PROD cluster!

```sh
db.dropDatabase()
```

Create a collection.

```sh
db.createCollection('collection_name')
```

Remove a collection and its index definitions.

```sh
db.coll.drop()
```

Rename a collection. The 2nd parameter drops the target collection if it exists.

```sh
db.coll.renameCollection("new_coll_name", true)
```

Return an array containing the names of all collections and views in the current database.

```sh
db.getCollectionNames()
```

Create collection with a $jsonschema validation.

```sh
db.createCollection("contacts", {
   validator: {$jsonSchema: {
      bsonType: "object",
      required: ["phone"],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         email: {
            bsonType: "string",
            pattern: "@mongodb\.com$",
            description: "must be a string and match the regular expression pattern"
         },
         status: {
            enum: [ "Unknown", "Incomplete" ],
            description: "can only be one of the enum values"
         }
      }
   }}
})
```

Add validation to a collection that was created without it, or change the validation schema of a collection with the `collMod` command.

```sh
db.runCommand( { collMod: "users",
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "username", "password" ],
         properties: {
            username: {
               bsonType: "string",
               description: "must be a string and is required"
            },
            password: {
               bsonType: "string",
               minLength: 12,
               description: "must be a string of at least 12 characters, and is required"
            }
         }
      }
   }
} )
```

Check the total size of a collection.

```sh
db.runCommand( { collStats.totalSize : "collection_name" } )
```

Return statistics that reflect the use state of a single database.

```sh
db.stats()
```

Return a document with information about the underlying system that the mongod or mongosh runs on.

```sh
db.hostInfo()
```

</details>

## 3. MONGODB CRUD

<details><summary>Insert. If the the collection does not exist, create operations also create the collection.</summary>

Insert a single document into a collection.

```sh
db.collection.instertOne({ key: "value" })
```

Insert multiple documents into a collection.

```sh
db.collection.insertMany({ key: "value" }, { key: "value" })
```

Insert multiple rows.

```sh
db.users.insertMany([
  {
    name: 'First User',
    age: 23,
    created_at: Date()
  },
  {
    name: 'Second User',
    age: 32,
    created_at: Date()
  },
  {
    name: 'Third User',
    age: 27,
    created_at: Date()
  },
])
```

</details>

<details><summary>Find.</summary>

Select and return all documents in a collection.

```sh
db.collection.find()
```

Find all documents that match the filter object.

```sh
db.collection.find({ key: "value" })
```

</details>

<details><summary>Update.</summary>

Update a single document within the collection based on the filter.

```sh
db.collection.updateOne({ key: "value" }, { $set: { key: "new_value" }})
```

Update all documents that match the specified filter for a collection.

```sh
db.collection.updateMany({ key: "value" }, { $set: { key: "new_value" }})
```

Replace a single document within the collection based on the filter.

```sh
db.collection.replaceOne({ name: "name" }, { name: "new_name" })
```

</details>

<details><summary>Delete.</summary>

Remove a single document from a collection.

```sh
db.collection.deleteOne({ key: "value" })
```

Remove all documents that match the filter from a collection.

```sh
db.collection.deleteMany({ age: {$lt:18} })
```

</details>

<details><summary>Bulk write operations.</summary>

Bulk write operations affect a single collection. By default, `bulkWrite()` performs ordered operations. To specify unordered write operations, set `ordered : false` in the options document. With an ordered list of operations, MongoDB executes the operations serially. If an error occurs during the processing of one of the write operations, MongoDB will return without processing any remaining write operations in the list. The following bulkWrite() example runs these operations on the pizzas collection:

- Adds two documents using insertOne.
- Updates a document using updateOne.
- Deletes a document using deleteOne.
- Replaces a document using replaceOne.

```sh
try {
   db.pizzas.bulkWrite( [
      { insertOne: { document: { _id: 3, type: "beef", size: "medium", price: 6 } } },
      { insertOne: { document: { _id: 4, type: "sausage", size: "large", price: 10 } } },
      { updateOne: {
         filter: { type: "cheese" },
         update: { $set: { price: 8 } }
      } },
      { deleteOne: { filter: { type: "pepperoni"} } },
      { replaceOne: {
         filter: { type: "vegan" },
         replacement: { type: "tofu", size: "small", price: 4 }
      } }
   ] )
} catch( error ) {
   print( error )
}
```

</details>

## 4. MONGODB OPERATORS

<details><summary>Query comparison operators.</summary>

Match values that are equal to a specified value.

```sh
$eq
```

Match values that are greater than a specified value.

```sh
$gt
```

Match values that are greater than or equal to a specified value.

```sh
$gte
```

Match any of the values specified in an array.

```sh
$in
```

Match values that are less than a specified value.

```sh
$lt
```

Match values that are less than or equal to a specified value.

```sh
$lte
```

Match all values that are not equal to a specified value.

```sh
$ne
```

Match none of the values specified in an array.

```sh
$nin
```

</details>

<details><summary>Field update operators.</summary>

Increment the value of the field by the specified amount.

```sh
$inc
```

Only update the field if the specified value is less than the existing field value.

```sh
$min
```

Only updates the field if the specified value is greater than the existing field value.

```sh
$max
```

Rename a field.

```sh
$rename
```

Set the value of a field in a document.

```sh
$set
```

Removes the specified field from a document.

```sh
$unset
```

</details>

<details><summary>Read modifiers.</summary>

Order the elements of an array during a `$push` operation. `cursor` is the end of the query, e. g. `db.users.find()`.

```sh
cursor.sort()
```

Specify the maximum number of documents the cursor will return.

```sh
cursor.limit()
```

Control where MongoDB begins returning results.

```sh
cursor.skip()
```

Append a specified value to an array.

```sh
cursor.push()
```

</details>

<details><summary>Aggregation operations.</summary>

Return a count of the number of documents in a collection or a view.

```sh
db.collection.count()
```

Return an array of documents that have distinct values for the specified field.

```sh
db.collection.distinct()
```

</details>

## 5. MONGODB INDEXES

<details><summary>Working with indexes.</summary>

Build an index on a collection.

```sh
db.collection.createIndex("index_name")
```

Remove a specified index on a collection.

```sh
db.collection.dropIndex("index_name")
```

Remove all indexes but the _id (no parameters) or a specified set of indexes on a collection.

```sh
db.collection.dropIndexes()
```

Return an array of documents that describe the existing indexes on a collection.

```sh
db.collection.getIndexes()
```

Rebuilds all existing indexes on a collection.

```sh
db.collection.reIndex()
```

Report the total size used by the indexes on a collection. It provides a wrapper around the totalIndexSize field of the collStats output.

```sh
db.runCommand( { collStats.totalIndexSize } )
```

</details>

## 6. MONGODB USER MANAGEMENT

<details><summary>Authenticate and manage users.</summary>

Authenticate a suer to the database from within the shell.

```sh
db.auth( <username> )
```

Return information for all users in the database.

```sh
db.getUsers()
```

Create a user.

```sh
db.createUser()
```

Delete a user.

```sh
db.dropUser()
```

Change a user's password.

```sh
db.changeUserPassword()
```

Grant a role and its privileges to a user.

```sh
db.grantRolesToUser()
```

Remove one or more roles from a user on the current database.

```sh
db.revokeRolesFromUser()
```

</details>

<details><summary>Manage roles and privileges.</summary>

Create a role.

```sh
db.createRole()
```

Delete a user-defined role associated with a database.

```sh
db.dropRole()
```

Return information for all user-defined roles in a database.

```sh
db.getRoles()
```

Assign privileges to a user-defined role.

```sh
db.grantPrivilegesToRole()
```

Remove the specified privileges from a user-defined role.

```sh
db.revokePrvilegesFromRole()
```

Update a user-defined role.

```sh
db.updateRole()
```

</details>
