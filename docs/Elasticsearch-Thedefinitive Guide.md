# Chapter 1.


```
Relational DB  ⇒ Databases ⇒ Tables ⇒ Rows ⇒ Columns
Elasticsearch  ⇒ Indices   ⇒ Types  ⇒ Documents ⇒ Fields

```

An Elasticsearch cluster can contain multiple indices (databases), which in turn contain multiple types (tables). These types hold multiple documents (rows), and each document has multiple fields (columns).


```
### INDEX VERSUS INDEX VERSUS INDEX
You may already have noticed that the word index is overloaded with several meanings in the context of Elasticsearch. A little clarification is necessary:

Index (noun)
As explained previously, an index is like a database in a traditional relational database. It is the place to store related documents. The plural of index is indices or indexes.

Index (verb)
To index a document is to store a document in an index (noun) so that it can be retrieved and queried. It is much like the INSERT keyword in SQL except that, if the document already exists, the new document would replace the old.

Inverted index
Relational databases add an index, such as a B-tree index, to specific columns in order to improve the speed of data retrieval. Elasticsearch and Lucene use a structure called an inverted index for exactly the same purpose.

By default, every field in a document is indexed (has an inverted index) and thus is searchable. A field without an inverted index is not searchable. We discuss inverted indexes in more detail in “Inverted Index”.

```

#Chapter 2.
A node is a running instance of Elasticsearch, while a cluster consists of one or more nodes with the same cluster.name that are working together to share their data and workload. As nodes are added to or removed from the cluster, the cluster reorganizes itself to spread the data evenly.

One node in the cluster is elected to be the master node, which is in chargeof managing cluster-wide changes like creating or deleting an index, or addingor removing a node from the cluster. 

### Cluster Health:  GET/_cluster/health

```

The status field provides an overall indication of how the cluster is functioning. The meanings of the three colors are provided here for reference:

green
All primary and replica shards are active.

yellow
All primary shards are active, but not all replica shards are active.

red
Not all primary shards are active.

```

### Add an Index

To add data to Elasticsearch, we need an index—a place to store related data. In reality, an index is just a logical namespace that points to one or more physical shards.


* A shard is a low-level worker unit that holds just a slice of all the data in the index. 

* a shard is a single instance of Lucene, and is a complete search engine in its own right. Our documents are stored and indexed in shards, but our applications don’t talk to them directly. Instead, they talk to an index.

* Shards are how Elasticsearch distributes data around your cluster. Think of shards as containers for data. Documents are stored in shards, and shards are allocated to nodes in your cluster.
 
* A shard can be either a primary shard or a replica shard

#Chapter 3.

In Elasticsearch, all data in every field is indexed by default.

### Document Metadata

* _index
  An index is like a database in a relational database; it’s the placewe store and index related data.
  
* _type
  
  In a relational database, we usually store objects of the same class in the same table, because they share the same data structure. For the same reason, in Elasticsearch we use the same type for documents that represent the same class of thing, because they share the same data structure.
  
* _id

### Retrieving a Document 
```
  GET /website/blog/123?pretty
  
```

#### NOTE
Adding pretty to the query-string parameters for any request, as in the
preceding example, causes Elasticsearch to pretty-print the JSON response
to make it more readable. The _source field, however, isn’t pretty-printed. Instead we get back exactly the same JSON string that we passed
in.

### Retrieving Part of a Document

```
GET /website/blog/123?_source=title,text
```

### Checking Whether a Document Exists

If all you want to do is to check whether a document exists—you’re not interested in the content at all—then use the HEAD method instead of the GET method. HEAD requests don’t return a body, just HTTP headers:

```
curl -i -XHEAD http://localhost:9200/website/blog/123
```

200 OK, 404 Not Found

### Creating a New Document

Remember that the combination of _index, _type, and _id uniquely identifies a document. So the easiest way to ensure that our document is new is by letting Elasticsearch autogenerate a new unique _id, using the POST version of the index request:

```
POST /website/blog/
{ ... } 
```

However, if we already have an _id that we want to use, then we have to tell Elasticsearch that it should accept our index request only if a document with the same _index, _type, and _id doesn’t exist already.

* The first method uses the op_type query-string parameter:

```
PUT /website/blog/123?op_type=create
{ ... }

```
* And the second uses the /_create endpoint in the URL:

```
PUT /website/blog/123/_create
{ ... }

```

If the request succeeds in creating a new document, Elasticsearch will return the usual metadata and an HTTP response code of 201 Created.

On the other hand, if a document with the same _index, _type, and _id already exists, Elasticsearch will respond with a 409 Conflict response code.

### Deleting a Document

```
DELETE /website/blog/123

```

###Dealing with Conflicts


* ####Pessimistic concurrency control:
Widely used by relational databases, this approach assumes that conflicting changes are likely to happen and so blocks access to a resource in order to prevent conflicts. A typical example is locking a row before reading its data, ensuring that only the thread that placed the lock is able to make changes to the data in that row.

* ####Optimistic concurrency control:
Used by Elasticsearch, this approach assumes that conflicts are unlikely to happen and doesn’t block operations from being attempted. However, if the underlying data has been modified between reading and writing, the update will fail. It is then up to the application to decide how it should resolve the conflict. For instance, it could reattempt the update, using the fresh data, or it could report the situation to the user.


### Optimistic Concurrency COntrol

Elasticsearch uses this _version number to ensure that changes are applied in the correct order. If an older version of a document arrives after a new version, it can simply be ignored.

We can take advantage of the _version number to ensure that conflicting changes made by our application do not result in data loss. We do this by specifying the version number of the document that we wish to change. If that version is no longer current, our request fails.


###Using Versions from an External System

If your main database already has version numbers—or a value such as timestamp that can be used as a version number—then you can reuse these same version numbers in Elasticsearch by adding version_type=external to the query string. Version numbers must be integers greater than zero and less than about 9.2e+18--a positive long value in Java.

```
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

### Partial Updates to Documents
