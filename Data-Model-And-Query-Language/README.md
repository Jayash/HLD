# Data Model And Query Language

Most applications are built by layering one data model on top of another. Each layer hides the complexity of the layers below by providing a clean data model. These abstractions allow different groups of people to work effectively.

### Relational model vs document model
The roots of relational databases lie in business data processing, transaction processing and batch processing.

The goal was to hide the implementation details behind a cleaner interface.

Not Only SQL has a few driving forces:

- Greater scalability
- preference for free and open source software
- Specialised query optimisations
-  Desire for a more dynamic and expressive data model

**With a SQL model, if data is stored in a relational tables, an awkward translation layer is translated, this is called impedance mismatch.**

JSON model reduces the impedance mismatch and the lack of schema is often cited as an advantage.

JSON representation has better locality than the multi-table SQL schema. All the relevant information is in one place, and one query is sufficient.

In relational databases, it's normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.

If the database itself does not support joins, you have to emulate a join in application code by making multiple queries.

The most popular database for business data processing in the 1970s was the IBM's Information Management System (IMS).

IMS used a hierarchical model and like document databases worked well for one-to-many relationships, but it made many-to-,any relationships difficult, and it didn't support joins.

### The network model

Standardised by a committee called the Conference on Data Systems Languages (CODASYL) model was a generalisation of the hierarchical model. In the tree structure of the hierarchical model, every record has exactly one parent, while in the network model, a record could have multiple parents.

The links between records are like pointers in a programming language. The only way of accessing a record was to follow a path from a root record called access path.

A query in CODASYL was performed by moving a cursor through the database by iterating over a list of records. If you didn't have a path to the data you wanted, you were in a difficult situation as it was difficult to make changes to an application's data model.

### The relational model
By contrast, the relational model was a way to lay out all the data in the open" a relation (table) is simply a collection of tuples (rows), and that's it.

The query optimiser automatically decides which parts of the query to execute in which order, and which indexes to use (the access path).

The relational model thus made it much easier to add new features to applications.

---

The main arguments in favour of the document data model are schema flexibility, better performance due to locality, and sometimes closer data structures to the ones used by the applications. The relation model counters by providing better support for joins, and many-to-one and many-to-many relationships.

If the data in your application has a document-like structure, then it's probably a good idea to use a document model. The relational technique of shredding can lead unnecessary complicated application code.

The poor support for joins in document databases may or may not be a problem.

If you application does use many-to-many relationships, the document model becomes less appealing. Joins can be emulated in application code by making multiple requests. Using the document model can lead to significantly more complex application code and worse performance.

### Schema flexibility
Most document databases do not enforce any schema on the data in documents. Arbitrary keys and values can be added to a document, when reading, clients have no guarantees as to what fields the documents may contain.

Document databases are sometimes called schemaless, but maybe a more appropriate term is schema-on-read, in contrast to schema-on-write.

Schema-on-read is similar to dynamic (runtime) type checking, whereas schema-on-write is similar to static (compile-time) type checking.

The schema-on-read approach if the items on the collection don't have all the same structure (heterogeneous)

- Many different types of objects
- Data determined by external systems

### Data locality for queries
If your application often needs to access the entire document, there is a performance advantage to this storage locality.

The database typically needs to load the entire document, even if you access only a small portion of it. On updates, the entire document usually needs to be rewritten, it is recommended that you keep documents fairly small.

Convergence of document and relational databases
PostgreSQL does support JSON documents. RethinkDB supports relational-like joins in its query language and some MongoDB drivers automatically resolve database references. Relational and document databases are becoming more similar over time.

### Query languages for data
SQL is a declarative query language. In an imperative language, you tell the computer to perform certain operations in order.

In a declarative query language you just specify the pattern of the data you want, but not how to achieve that goal.

A declarative query language hides implementation details of the database engine, making it possible for the database system to introduce performance improvements without requiring any changes to queries.

Declarative languages often lend themselves to parallel execution while imperative code is very hard to parallelise across multiple cores because it specifies instructions that must be performed in a particular order. Declarative languages specify only the pattern of the results, not the algorithm that is used to determine results.

Declarative queries on the web
In a web browser, using declarative CSS styling is much better than manipulating styles imperatively in JavaScript. Declarative languages like SQL turned out to be much better than imperative query APIs.

### MapReduce querying
MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularised by Google.

Mongo offers a MapReduce solution.

The map and reduce functions must be pure functions, they cannot perform additional database queries and they must not have any side effects. These restrictions allow the database to run the functions anywhere, in any order, and rerun them on failure.

A usability problem with MapReduce is that you have to write two carefully coordinated functions. A declarative language offers more opportunities for a query optimiser to improve the performance of a query. For there reasons, MongoDB 2.2 added support for a declarative query language called aggregation pipeline

## Graph-like data models
If many-to-many relationships are very common in your application, it becomes more natural to start modelling your data as a graph.

A graph consists of vertices (nodes or entities) and edges (relationships or arcs).

Well-known algorithms can operate on these graphs, like the shortest path between two points, or popularity of a web page.

There are several ways of structuring and querying the data. The property graph model (implemented by Neo4j, Titan, and Infinite Graph) and the triple-store model (implemented by Datomic, AllegroGraph, and others). There are also three declarative query languages for graphs: Cypher, SPARQL, and Datalog.

Property graphs
Each vertex consists of:

- Unique identifier
- Outgoing edges
- Incoming edges
- Collection of properties (key-value pairs)

Each edge consists of:

- Unique identifier
- Vertex at which the edge starts (tail vertex)
- Vertex at which the edge ends (head vertex)
- Label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)
Graphs provide a great deal of flexibility for data modelling. Graphs are good for evolvability.
---
- Cypher is a declarative language for property graphs created by Neo4j
- Graph queries in SQL. In a relational database, you usually know in advance which joins you need in your query. In a graph query, the number if joins is not fixed in advance. In Cypher :WITHIN*0... expresses "follow a WITHIN edge, zero or more times" (like the * operator in a regular expression). This idea of variable-length traversal paths in a query can be expressed using something called recursive common table expressions (the WITH RECURSIVE syntax).
---

### Triple-stores and SPARQL
In a triple-store, all information is stored in the form of very simple three-part statements: subject, predicate, object (peg: Jim, likes, bananas). A triple is equivalent to a vertex in graph.

### The SPARQL query language
SPARQL is a query language for triple-stores using the RDF data model.

### The foundation: Datalog
Datalog provides the foundation that later query languages build upon. Its model is similar to the triple-store model, generalised a bit. Instead of writing a triple (subject, predicate, object), we write as predicate(subject, object).

We define rules that tell the database about new predicates and rules can refer to other rules, just like functions can call other functions or recursively call themselves.

Rules can be combined and reused in different queries. It's less convenient for simple one-off queries, but it can cope better if your data is complex.
