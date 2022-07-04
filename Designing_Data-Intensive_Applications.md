# Designing Data-Intensive Applications

# Table Of Contents
  - [Part 1 - Foundations of Data Systems ](#part-1---foundations-of-data-systems)
  - [Part 2 - Distributed Data](#part-2---distributed-data)
  - [Part 3 - Derived Data](#part-3---derived-data)


# Part 1 - Foundations of Data Systems

## Chapter 1: Reliable, Scalable, and Maintainable Applications
- Many applications are `data-intensive` instead of `compute-intensive`
- Standard building blocks for a data-intensive application:
  - database
  - cache - speed up reads
  - search index - support user search by key word or filter
  - stream processing - send messages to another process
  - batch processing
### Thinking About Data Systems
- Reliability - System should work correctly in the face of `adversity`
- Scalability - There should be ways for the system to grow
- Maintainability - Many different people will work on the system both to maintain behavior and to add new behavior.  They should be able to do so productively.
### Reliability
- Things that can go wrong are called `faults`.  We want `fault-tolerant` or `resilient` systems
- `fault` is different from `failure`.  Fault is when one piece misbehaves, failure is when the system misbehaves.  Cope with faults to minimize failures.
- Increasing `faults` may be desirable to reduce `failures`, a la Chaos Monkey
#### Hardware Faults
- In modern eco system, we want to move toward architecture that can tolerate the loss of entire machines
#### Software Errors
- No simple solution here, need lots of things
- testing, process isolation, allow crash/restart, measure, monitor, analyze behavior
- Example: If a system should guarantee something like equal number of incoming messages and outgoing messages, it can monitor itself and alert when a discrepancy arises
#### Human Errors
- Try to make it hard to "do the wrong thing"
- Decouple the place where humans make mistakes from the place where they cause `failure`. e.g., create sandbox environments
- Test thoroughly, from unit test to system integration test, to manual test
- Allow for Easy Recovery from errors
- Set up good monitoring, aka `telemetry`
- Have good management practices
### Scalability
#### Describing Load
- `load parameters` describe load
- Actual parameters depend on architecture
- Twitter Example 
  - 2 key operations: `post tweet` and `home timeline`
  - 12k writes per second is easy, but scaling challenge comes from `fan-out`.  Each user follows many people, and each user is followed by many people.
  - Two options to handle this:
    1. Posting tweet just inserts one entry.  Then when querying home timeline, you query for all the tweets from everyone they follow
    2. Maintain a cache of each user's home timeline, and update everyone's timeline when new tweet gets created.  Then lookups are much faster
  - Twitter went from approach 1 to 2, and is now moving toward a hybrid approach
#### Describing Performance
- Two ways to think about it:
  - If you increase a `load parameter` and dont change system resources, how does performance change
  - If you increase a `load parameter`, how much do you need to increase resources to keep performance unchanged
- batch processing systems like Hadoop - `throughput`
- Online systems - `response time`
  - `response times` vary, so think of them as a `distribution` of times to measure
- Rather than just use `average response time`, its better to look at `percentiles`
- High percentiles are called `tail latencies` - Important beause they actually show users' experience
- Example - Amazon measures `99.9th percentile` because that is likely to reflect the power user's experience, aka the account with lots of data
- Percentiles are often used in `Service Level Objectives` and `Service Level Agreements`
- We can look at some percentiles to get an idea of performance: `p50`, `p95`, `p99`, `p999`
#### Approaches for Coping with Load
- `Scale Up` vs `Scale Out`
- In most cases, some mix of both can be good.  Using several fairly powerful machines can be simpler and cheaper than a lot of small virtual machines
- Stateless services are easy to distribute, but stateful ones are not
- Conventional wisdom is to try to keep db on a single node, but this may change, lots of new products to try to solve this
### Maintainability
- `Operability` - Make it easy for operations teams to keep system running well
- `Simplicity` - Remove complexity of the system.  Different from creating simple UI
- `Evolvability` - Make it easy to change the system in the future
#### Operability
Operations teams do:
- Monitoring health and restore service
- Track the cause of problems
- Keep software up to date, including security patches
- Be aware of system interactions to avoid problematic changes ahead of time
- Capacity Planning
- Establish good practices for deployment, config management, etc
- Perform complex maintenance tasks like app migrations
- Maintain security of the system as config changes are made
- Define processes that make operations predictable and keep things stable
- Preserve organizational knowledge/institutional knowledge
#### Simplicity: Managing Complexity
- Use `abstraction` to remove accidental complexity.  Hide a bunch of details behind a clean facade
- Finding good abstractions is hard
#### Evolvability: Making Change Easy
- `Agile` and `TDD` try to provide tools for dealing with changes
- What can we do with larger scale changes, like Twitter Architecture

## Chapter 2: Data Models and Query Languages
- Data models are important because they change how we `think about the problem` we are solving
- Key question: How is the datamodel `represented` in terms of the next lower layer
  - e.g.application takes real world and turns it into objects/data structures; then you turn them into json/xml/tables and store into db; then db stores that onto disk; etc etc
- Lets cover some common data models for storage and querying
  - relational model
  - document model
  - graph-based
### Relational Model Versus Document Model
- Relational model was proposed in 1970 by Edgar Codd
- Became the dominant model.  Others like hierarchical model and network model were around but relational is far more popular
#### The Birth of NoSQL
- Got hot in 2010s
- Key drivers:
  - Greater scalability than relational dbs
  - Preference for free and open source software
  - Specialized queries that don't work well in relational model
  - Frustration with strict schemas
Relational databases will probably remain in use alongside various NoSQL stores - `polyglot persistence`
#### The Object-Relational Mismatch
- Translating between code objects and relational storage causes `impedance mismatch`
- One-toMany relationship is common
  - Traditionally (pre SQL:1999) we normalize into separate tables with foreign keys
  - Newer SQL standards have structured datatypes and XML data, or JSON datatype.  Supported by some products
  - Alternatively we can encode an object into a BLOB and store it as a text type or something like that
- Some things like a resume could be better represented as a JSON object
  - This can reduce `impedance mismatch` but may have other issues
  - JSON representation has better `locality`; all data comes from a single place
#### Many-to-One and Many-to-Many Relationships
- In relational model, we often wind up having lookup tables with a unique ID and then the value, rather than just the value
- This normalization makes it easy to deduplicate data and to join data
- Document databases dont have great support for joins, so responsibility shifts to the application
- Data model changes in a document model may cause problems later on in terms of joins
  - e.g. In a resume with an `organization` section, instead of a hardcoded name we can have a link to an entity or object or URL
  - e.g. PersonA writes recommendation for PersonB on their resume app.  Then PersonB's resume should have link to PersonA's profile to show the details so its automatically updated when PersonA edits their info.
#### Are Document Databases Repeating History?
- Before relational model, IBM's `Information Management System` was popular.  It used `hierarchical model`
- Good for one-to-many relationships, but not for many-to-many and no joins
##### The network model
- Network Model is also called CODASYL model
- Generalized version of `hierarchical model`.  Each record can now have multiple parents
- Links between records are similar to pointers, and you access a record by following an `access path` to it
- Tracking many to many joins can become very difficult
- Changing queries or data model becomes super difficult because you need to rewrite the access paths
##### The relational model
- Just lay the data into `relation` and `tuples`
- Query optimizer decides how to access the data
- Query Optimizers are more difficult to implement once, but you implement the general solution once and everyone can use it more easily than the network model
##### Comparison to document databases
- Document databases go back to hierarchical model by storing nested records directly in parent record
- But for many-to-one and many-to-many, they use references
  - Relational db uses `foreign key`
  - Document model uses `document reference`
#### Relational Versus Document Databases Today
- Pros for Document model: schema flexibility, better performance from locality, and less impedance mismatch
- Pros for Relational model: better joins and relationships
##### Which data model leads to simpler application code?
- If data matches document model it can be easier
- `shredding` for relational dbs can be complex
- If you have many-to-many relationships, document model may be a pain
##### Schema flexibility in the document model
- Instead of `schemaless`, `schema-on-read` is more accurate to describe document model, a opposed to `schema-on-write`
- `schema-on-read` can help if each item in the collection has different structure, ie heterogenous data
##### Data locality for queries
- Data locality of document model is nice if you usually are querying the majority of the document and not small pieces of it
- Document updates usually require full re-write of the document, making smaller documents recommended.  But this is a drawback/limitation of document model
##### Convergence of document and relational databases
- Many relational dbs started to support XML and JSON
- Many document dbs like RethinkDB support relational-like joins
### Query Languages For Data
- SQL is `declarative` as opposed to IMS and CODASYL which were `imperative`
- Declarative approach can be easier to parallelize and speed up
#### Declarative Queries on the Web
- Thought experiment: think about trying to change style of elements in a html page
- css or XSL would follow the `declarative` approach and are simple
- we could also try to write custom JavaScript to search the DOM and change style based on that, which would be `imperative` approach
  - would be super messy and hard to maintain
#### MapReduce Querying
- Somewhere in between `declarative` and `imperative`
- Compare a postgres SQL query to select some data about shark observations to MongoDB MapReduce feature:
```
db.observations.mapReduce(
    function map(){
        var year = this.observationTimestamp.getFullYear();
        var month = this.observationTimestamp.getMonth() + 1;
        emit(year + "-" + month, this.numAnimals);
    },
    function reduce(key, values){
        return Array.sum(values);
    },
    {
        query: { family: "Sharks" },
        out: "monthlySharkReport"
    }
);
```
- map function is called once for each document that matches the query
  - emits a key and a value
- reduce function is called *once* for all key-pair values that share a key
- query - declarative specification of a filter
- out - tells the db to write to a collection called monthlySharkReport
- Syntax like this does put more responsibility on the user to write the corresponding map and reduce functions
- MongoDB introduced the `aggregation pipeline` declarative pipeline to help with this
### Graph-Like Data Models
- For data with lots of many-to-many relationships, graphs are more appropriate
- Graph data examples:
  - Social graphs - connections between people
  - Web graph - links between web pages
  - Road or rail networks
- Graph databases can store heterogenous data
  - Facebook has a single graph with many types of vertices like people, locations, events, checkins and comments
  - edges can represent friends, checkins that happened in a location, who commented on a post, etc
- Create an example of graph data, with 2 people from two locations who now both live in London.  Look at various ways it is represented
  - `Property Graph`
  - `Triple-Store Model`
- Query languages:
  - `Cypher`
  - `SPARQL`
  - `Datalog`
#### Property Graphs
- Each `vertex` has
  - unique ID
  - set of outgoing edges
  - set of incoming edges
  - collection of properties
- Each `edge` has
  - unique ID
  - `tail vertex` - where the edge starts
  - `head vertex` - where the edge points to
  - label to describe the relationship
  - collection of properties
- Example of how to represent a `property graph` in Relational DB
```
CREATE TABLE vertices (
    vertex_id integer PRIMARY KEY,
    properties json
);

CREATE TABLE edges (
    edge_id integer PRIMARY KEY,
    tail_vertex integer REFERENCES vertices (vertex_id),
    head_vertex integer REFERENCES vertices (vertex_id),
    label text,
    properties json
);

CREATE INDEX edge_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```
- Property graph model is flexible for cases where data isn't homogenous
  - e.g. if one person is born in a country with states and another person born in a country with city as the place of birth
#### The Cypher Query Language
- `cypher` is declarative query language for property graphs, from Neo4j
- Sample Syntax:
```
MATCH
  (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
  (person) -[:LIVES_IN]->() -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```
- This query will do:
  - find all vertex that has a BORN_IN relationship to another vertex and then eventually has a WITHIN relationship to location united states
  - find all vertex that has a LIVES_IN relationship to another vertex and then eventually a WITHIN relationship to Location Europe
  - Refer to any matching vertexes as *person* variable, and return *person.name*
#### Graph Queries in SQL
- Theoretically we can try to support Cypher like queries in Relational db, but obviously not so easy
- One big challenge is the unknown number of edges to traverse to check if clause is correct
- We can use `recursive common table expression` to try to solve that
- Skip big example, but a 4 line cypher query took 29 lines of SQL to do
#### Triple-Stores and SPARQL
- `Triple Store` uses different words/concepts from `Property graph` approach
- All info has 3 parts: `subject`, `predicate`, `object`, e.g. *Jim likes bananas*
- Object in a graph can be equivalent of 2 ideas from property graph
  - some kind of data value, in which case the triplet is like a property of the vertex
    - e.g. Lucy, age, 33
  - Another vertex, in which case the triplet is like an edge between two vertexes
    - e.g. Lucy, marriedTo, Alain
##### The semantic web
- Triple stores are closely associated to `semantic web`, but not specifically related
- `semantic web` is an idea to publish information about what is on the web using `Resource Description Framework`, aka `RDF`
##### The RDF data model
- `Turtle` is a language to do human-readable representation for RDF
- `Apache Jena` can convert between various formats like Turtle or XML
##### The SPARQL query language
- `SPARQL` is a query language for triple-stores using RDF model
- Example:
```
PREFIX : <urn:example:>

SELECT ?personName WHERE {
  ?person :name :/personName.
  ?person :bornIn / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
```
#### The Foundation: Datalog
- `Datalog` is older than SPARQL or Cypher, but less well known
- Used in a few systems such as Datomic, or Cacalog which is an implementation for querying large datasets in Hadoop
### Summary
- Historically data started out being represented as one big tree, but many-to-many relationships wer challenging
- Relational model got popular
- More recently NoSQL tries to solve new challenges, and there are two large categories
  - Document databases
  - Graph databases
- Relational, document, and graph model are all widely used today

## Chapter 3: Storage and Retrieval
- High level, database does 2 things: store data, and retrieve it when we need it
### Data Structures That Power Your Database
- Example simple database :
```
#!/bin/bash

db_set (){
    echo "$1,$2" >> database
}

db_get (){
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```
- Allows us to save data like `db_set 42 '{"name":"San Francisco","attractions":["Golden Gate Bridge"]}'`
- Allows us to fetch data like `db_get 42` 
- Write performance would be *good* since we only append to a file
- Read performance would get very *bad* as the file grows huge since we need to grep
- This database doesn't overwrite old data, which is why db_get must `tail -n 1`
- `indexes` can help with searching as data gets large
- Read performance gets better, but write overhead to keep metadata up to date gets higher
#### Hash Indexes
- Consider a data structure to *index* key-value data like the previous section
- Use a `hash map` or `dictionary` data structure to create this index
  - key will be the key of the data
  - value of the map will be the byte offset where that entry starts in the file
  - `Bitcask` uses this general approach, and requires that all keys fit in RAM
- What happens if the file gets too huge?
  - We should *close* the current segment file and then start writing to a new segment file
  - When we do this, we should do `compaction` and `merging` to reduce the number of files and overall size of data
    - `compaction` means go through the file and throw away any duplicate keys since only the last entry for a key is current data
    - `merging` means once you do compaction, each file is smaller, so we can put them together into a single file
    - The *frozen* segments won't be updated anymore, so we can just serve any read requests using the original copy until merge and compaction is completed in the background
  - Now we have a bunch of these segment files, and each one can have its own Hash Index
  - Any read request can search the Hash Index of each segment in order from newest file to oldest
  - Writes are performant usually in single thread since its append only
- Some real world optimizations required on this idea
  - File Format - Instead of csv, use a binary format that encodes length of string
  - Deletions - Use `tombstone` data records to signal that a record should be deleted and this can be processed during merging
  - Crash recovery - Take snapshots of hash maps to disk to help with recovery
  - Partially written records - Use checksums to ensure that entries aren't partially written and ignore any partially written entries
  - Concurrency - write with single thread, read with many threads
- Drawbacks - 
  - This only works well if hash maps can stay in memory
  - Cant really do any range searches
#### SSTables and LSM-Trees
- Make one key change to the above model: require all data within a file to be sorted by key
- Call this a `Sorted String Table` aka `SSTable`
- Advantages compared to previous `log segement with hash index` approach
  - `Merging` files can be done more easily using a `mergesort` kind of approach where the files are read in side by side
  - Indexes can now be `sparse index` and just index some subset of the data in a file, and you can rely on the sorted keys to find your desired entry from there
  - We can group records into a block and compress it before writing to disk to save space on disk
##### Constructing and maintaining SSTables
- Incoming writes are in random order so how do we keep data sorted
- Use a datastructure like a `red-black tree` or `AVL tree` for incoming writes. aka `memtable`
- When the `memtable` hits some size threshold, write it out to disk as a SSTable file
- Reads can start with the `memtable` then go to newest on-disk segment, then to older files
- Run periodic merge-compact on the log files
- Crash recovery - `memtable` is in memory so keep a log on disk to help with recovery
##### Making an LSM-tree out of SSTables
- This SSTable with memtables approach is called `Log-Structured Merge-Tree` or `LSM Tree`
- `Lucene` uses similar approach for its term dictionary
##### Performance optimizations
- Searching LSM-Tree for a non-existent key can be expensive; use `bloom filters` to help check if keys exist or not up front
- Different options for how to decide on the order and timing of when to do merging and compaction; `size-tiered` and `leveled compaction`
#### B-Trees
- Use `pages` to store the b-tree structure
- Each page can contain a bunch of key-value pairs where the value is a pointer to another page
- Start at root page, find the desired range, and follow the pointer to the next page until you get to the leaf page where the data sits or we actually get finally a pointer to the data page
- `branching factor` is the number of entries per page of the b-tree structure
- `Updates` are simple, just find the value in leaf node and change the value
- `Insertions` are harder, find the page where the data belongs, and insert if it fits.  Otherwise you need to do a page split
- B-Trees are `balanced` and always have a *depth* of `O(log n)`
##### Making B-Trees reliable
- Unlike LSM-Tree, we often overwrite data in the tree
- Use a `write-ahead log` aka `WAL` to help with consistency and durability
- `concurrency` is handled via `latches` which are light-weight locks
##### B-tree optimizations
- Instead of changing pages along with a WAL, some dbs can use `copy-on-write` and create a new version of the page and then update references to point to the new page
- Sometimes some of the intermediate layers can save space and pack more keys into a page with abbreviations
- Sometimes B-Tree can have additional pointers in the leaf node to neighbor pages to help with scans and range queries
#### Comparing B-Trees and LSM-Trees
- LSM trees are faster for writes, B-Trees are faster for reads
##### Advantages of LSM-trees
- `write amplification` is the effect where one write to the db requires multiple writes to disk
- Both LSM-trees and B-trees have `write amplification` but LSM-trees are less, and also more sequential which is good on magnetic drives
- LSM-trees can be compressed better than B-trees
##### Downsides of LSM-trees
- compaction can have performance impact at the higher percentiles
- compaction needs to be tuned to keep up with volume of incoming data
#### Other Indexing Structures
##### Storing values within the index
- Index can either choose to have a pointer to the data page, or keep the data in the index
  - pointer approach would then point to a `heap file` where the data lives
  - pointer approach is a `non clustered index`
  - data in index approach is `clustered index`
- Alternative to these would be `covering index` or `index with included columns`
##### Multi-column indexes
- What if you want to search according to multiple keys
- ```SELECT * from restaurants where latitude > x and latitude < y and longitude > a and longitude < b```
  - This example cannot benefit from any one B-tree of LSM-tree index
  - `R-tree` datastructure can be used
##### Full-text search and fuzzy indexes
- How to handle situations to search for similar terms or typos
- `edit distance` refers to how many letters are added, removed or replaced
- Lucene uses something like a `trie` and can then transform into a `levenshtein automaton` to search for words within edit distance
##### Keeping everything in memory
- `memcached` is an in-memory key-store value and is for caching only, data is lost on restart
- Other in-memory databases can try to create durability by creating snapshots to disk for recovery
- A lot of performance gain isn't about disk speed vs memory speed, but rather because they dont have to encode data into  formats for disk
  - `redis` offers database-like interface to datastructures like queues and sets
- New research into creating in-memory databases larger than the actual memory size
  - `anti-caching` would evict least recently used data to disk and load back when accessed
### Transaction Processing or Analytics?
- `OLTP` vs `OLAP`
- Some Key features
  - OLTP reads small number of records while OLAP aggregates over large number of records
  - OLTP does random access writes while OLAP does bulk imports or event stream
  - OLTP is used by end users or web apps, OLAP is used by internal analyst for decision support
#### Data Warehousing
- OLTP database is often relied on for realtime usage so usually avoid doing big queries on it
- `data warehouse` is a separate db for analysts to query with read-only data
- `Extract Transform Load` aka `ETL` is the process of extracting data from OLTP databases, transforming it, then loading it into a warehouse
##### The divergence between OLTP databases and data warehouses
- Both OLTP and OLAP can use relational, but internals are different due to different queries
- Vendors are starting to specialize into one or the other
#### Stars and Snowflakes: Schemas for Analytics
- `star schema` is the main model in data warehouses
- `fact table` represents an event that occurred at some time
- `fact table` has foreign key references to `dimension tables` that represent the `who, what, where, when, how, why` of the event
- `snowflake schema` is a variation on `star schema` where the `dimensions` are broken into `subdimensions`
  - e.g. if we have a dimension table like `dim_product` table, we can then further normalize that to have `foreign keys` to point to another dimension table for the brand and the category
  - This is more normalized than star schema
  - But less simple to work with if not strictly needed
### Column-Oriented Storage
- In `star schema` the fact table can become enormous
- Most queries should not be doing `select *` but rather only query a few columns
- Instead of organizing data by row, we can organize by column
#### Column Compression
- Columns with similar data can be compressed a lot
- One strategy can be `bitmap encoding`
  - Column has `n` distinct values
  - For **each** distinct value in the column, create a `bitmap` to show which rows have that value
  - If there are more than a few distinct values, we can then do `run-length encoding` to further compress the column
  - Example:
    - Original data: 69, 69, 69, 69, 74, 31, 31, 31, 31, 29, 30, 30, 31, 31, 31, 68, 69, 69
    - Create a bitmap for each value, so for 69 we have:
      - 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1
    - And then when we run-length encode it, it will become:
      - 0, 4, 12, 2
##### Memory bandwidth and vectorized processing
- The bandwidth of memory being loaded into CPU can become a major performance bottleneck with huge data
- Column oriented data can make it easy to fit a lot of data into CPU L1 cache and iterate over it without making a function call for each entry
- This is called `vectorized processing`
#### Sort Order in Column Storage
- Usually the order of rows wont matter, but sometimes we may know that we need some order to support certain queries
- You need to sort all the columns in the same order or you cannot reconstruct rows
- Example of storing in order: Maybe use a date_key as the first column of a sort key, and then it is quick to run a query that searches based on that date
- Sorting columns can also help optimize compression
##### Several different sort orders
- `C-store` and `Vertica` have a clever extension where they need to replicate data to multiple machines, so they sort the data differently on each replica
- Now you can use the version that is sorted to best handle your query
#### Writing to Column-Oriented Storage
- Writes are much harder than B-trees
- We can use similar approach from `LSM-trees for writes`
  - Handle all inserts via an in-memory store that keeps data sorted
  - Then prepare to write that to disk
#### Aggregation: Data Cubes and Materialized Views
- Separate from column-oriented data, we can use `materialized view` to help cache and save the results of some queries on disk
- Commonly used for `data cube` or `OLAP cube`
  - Try to pre-compute aggregates grouped by multiple dimensions
  - For example, imagine we have sales with `date` and `product`
    - We can create the `data cube` where each cell shows the value for each `date-product` combo
    - e.g. 20140101/product_id 32 = 149.60, 20140101/product_id 33 = 31.01, etc
    - Now its easy to lookup any individual cell, or to get something like all sales on 20140101 or all sales of product_id 32 over certain range of dates
  - Still good to keep the raw data to help with other drill-downs
### Summary
- Reviewed OLTP vs OLAP
- OLTP has 2 main families of engines
  - LSM-trees
  - B-trees and pages
- When doing analytics, rather than finding individual entries, its more important to scan lots of data
  - encoding data compactly is helpful, so column-oriented solutions help achieve that goal

## Chapter 4: Encoding and Evolution
- We need to support `evolvability` of applications
- We often do not make a change instantaneously; rolling upgrades or client-driven upgrades can mean we run multiple versions of software
- Need to maintain
  - Backward compatibility
  - Forward compatibility
### Formats for Encoding Data
- In memory data structures like objects, structs, lists, etc are designed for efficient access and manipulation by cpu
- When writing data to file or over network, you need to encode that into a sequence of bytes
- `encoding` or `serialization` or `marshalling` is translation from in-memory to byte-sequence
- Opposite is `decoding` or `deserialization` or `unmarshalling`
#### Language-Specific Formats
- Many languages have built in libraries for encoding and decoding, like Java.io.Serializable
- But they have problems
  - encoding and decoding are language specific
  - Decoding logic needs to create arbitrary classes and this can be a security vulnerability
  - Data versioning isn't well supported
  - Performance can be bad
- Generally avoid built-in encoding
#### JSON, XML, and Binary Variants
- JSON, XML or CSV are decent general purpose formats, but not perfect
##### Binary encoding
- If data volumes are huge, size of the payload can be a problem so binary encodings can help
- JSON has MessagePack, BSON, BJSON, UBJSON, BISON, Smile, etc
- XML has WBXML, Fast Infoset
- Since schema is not required, the encoded data needs to include the field names too
- Sample message
```
{
  "userName": "Martin",
  "favoriteNumber": 1337,
  "interests: ["daydreaming", "hacking"]
}
```
- Total length is 81 bytes
- Encoding example using `MessagePack` brings this down to 66 bytes, so small improvement
- `83|a8|75|73|65|72|4e|61|6d|65`
  - 0x83 - Top 4 bits are 0x80 and that means we have an object.  Bottom 4 bits are 3, meaning we have 3 fields
  - a8 - Top four bits = 0xa0 which means string, and bottom 4 bits mean we have length of 8
  - Next 8 bytes are ASCII for "userName"
#### Thrift and Protocol Buffers
- `Apache Thrift` and `Protocol Buffers` are both binary encoding libraries
- Thrift was from Facebook, Protobuf was from Google
- Both require a `schema` for the data to encode
- Example schema in ProtoBuf
```
message Person {
  required string user_name       = 1;
  optional int64  favorite_number = 2;
  repeated string interests       = 3;
}
```
- Code generation tool can read a schema and produce classes for it
- Once we have a schema, message becomes shorter because instead of including field name, we can replace it with field tag just to show the number of each field
- Example:
  - userName: Martin can now become
  - 0b|00 01|00 00 00 00 06|4d 61 72 74 69 6e
  - 0b = 11 = type string
  - 00 01 = 1 = field 1 according to our schema
  - 00 00 00 00 06 = length of 6
  - The rest is ascii for Martin
- Thrift has 2 encoding formats, `BinaryProtocol` and `CompactProtocol`
- Protocol Buffer has another format
- all have similar concept but different details about how field tags and data types are stored
##### Field tags and schema evolution
- tag *name* can change because we don't rely on it for the data tranfer
- tag *number* can never change because all existing data would become unreadable
- Adding new fields to schema
  - No problem to add new fields, but must be *optional* field or have default value
  - Forward compatibility is OK - old code can just ignore fields it doesn't know about
  - Backward compatibility is OK - new code can still read old data because tag names wont change
- Removing fields
  - Cannot remove required fields
  - Never re-use a tag once its removed to avoid backward compatibility issues
##### Datatypes and schema evolution
- Sometimes works
- If we change 32bit int to 64bit int
  - Backward compatibility should be fine because new code has no problem reading 32bit data
  - but forward compatibility can cause issues if it enounters 64 bit values
#### Avro
- Another binary encoding format
- Started from Hadoop
- Has a human readable schema called Avro IDL and a JSON based schema for machine readable version
- Most compact encoding protocol
- Does not include datatypes, only length of field and data
- You must have *exact* match schema available to read in the data and read in the field tags according to the schema only
##### The writer's schema and the reader's schema
- Avro is designed to support some schema mismatch; instead they must be compatible
- Avro will look at `reader's schema` and `writer's schema` and compare
  - Field orders can mismatch because code can just map them with both schemas
  - field missing in reader's schema gets ignored
  - field missing in writer's schema can get filled in with default value
##### Schema evolution rules
- You can only add or remove fields that have a default value
##### But what is the writer's schema?
- Multiple ways to solve how to send writer's schema to the reader depending on use-case
- Large file with lots of records: Just send the writer schema once along with a lot of data
- Database with individually written records: Record the writer schema version along with each record so it can be retrieved for decoding as needed
- Sending records over network - Negotiate a schema version during connection setup
  - e.g. Avro RPC Protocol
- We can track schema versions with numbering or hash of schema
##### Dynamically generated schemas
- Avro doesn't have tag numbers in the schema.  This is the key difference that makes it more friendly for auto generated schemas
- Example use-case: If you want to dump data from a database and encode it into Avro
  - You can generate an Avro schema from the relational schema
  - If db schema changes, you can generate new Avro schema and use that
  - reader's schema can just look at writer's schema and still decode the messages
  - In Thrift or Protobuf, this would be harder as field tags need to be assigned by hand
##### Code generation and dynamically typed languages
- Thrift and Protobuf rely on code generation
  - This is useful for statically typed languages
- Avro can be used without code generation
  - Use the `object container file` which embeds writer's schema and you can look at the data just like JSON data
  - This is useful with dynamic typed data processing languages like `Apache Pig`
#### The Merits of Schemas
- More compact than various binary JSON variants since they dont have field names
- schema is valuable as documentation as you know the schema is up to date for decoding purposes
- Keep a database of schemas allows you to check forward and backward compatibility before making changes
- Auto generated code in statically typed languages is useful
- We can get similar flexibility to the `schema-on-read` JSON databases, with better guarantees about data
### Modes of Dataflow
#### Dataflow Through Databases
- We can think of read and write to database similarly to the encode data/decode data problem
- We may have different versions of software reading/writing from db
- What happens if old release reads and updates a data entry that was written by new version
  - Easy to "lose" the fields that only exist in new version
  - Make sure the app takes care of this
##### Different values written at different times
- Key point: `data outlives code`
##### Archival storage
- Sometimes you may create data snapshots for backup or loading into data warehouse
- Format like Avro with object container files may be a good fit
- This is also good time to encode data into column-oriented format like `Parquet`
#### Dataflow Through Services: REST and RPC
- client server concept
- server can also be a client to another server - microservices or service oriented architecture
- service oriented architecture should allow each service to be independently released without coordination
  - make sure forward and backward compatibility are not a problem
##### Web services
- If HTTP is the underlying protocol it is called a `web service`
- Two popular approaches: `REST` and `SOAP`
- `REST` is not a protocol but rather a design philosophy
  - simple data formats
  - use urls for identifying resources
  - use HTTP features for cache control, authentication and content type negotiation
- `SOAP` is XML based protocol
  - Tries to be independent of HTTP
  - Instead uses its own standards: `web service framework` or `WS-*`
  - Web Services Description Language aka WSDL describes the SOAP api
##### The problems with remote procedure calls (RPCs)
- `Remote Procedure Call` tries to make a request to a remote network service look like calling a function inside your programming language
- Problems with this:
  - network problems are more common than a problem inside your own process
  - network request can timeout which is not a thing that happens with local calls
  - retry logic on failure might actually trigger the same thing multiple times on the target service
  - timing is much less consistent
  - Passing data requires encoding, cant just reference a variable in memory
  - data type mapping can be challenging if languages mismatch
##### Current directions for RPC
- There are newer generations of RPC frameworks using the encodings mentioned earlier
- No need to pretend the RPC is just like a local call
- Finagle and Rest.li offer `futures`
- gRPC supports `streams`
- support for `service discovery`
- Custom RPC can have better performance compared to JSON over REST
- But not as easy to experient with 
- RPC may become specialized for requests between services within one org within a datacenter
##### Data encoding and evolution for RPC
- We can assume server always gets updated first, then client
- Only focus on backward compatibility for requests and forward compatibility for responses
- Compatibility challenges will depend on underlying encoding scheme
  - gRPC uses protobuf
  - Avro RPC will use Avro
  - SOAP uses xml
  - JSON needs to have its own custom considerations
- API versioning has no single standard, so can be done in various ways
#### Message-Passing Dataflow
- Advantages of `message brokers`
  - Can act as a buffer if recipient is unavailable
  - redeliver lost messages if process crashes
  - sender doesn't need to know recipient address
  - decouple sender and receiver
##### Message brokers
- General pattern:
  - one process sends message to a `queue` or `topic`
  - broker makes sure that message goes to one or more `consumer` or `subscriber`
  - Topics provide one-way data flow
##### Distributed actor frameworks
- `actor model` - programming model for concurrency in a single process
- Don't directly deal with threads, but encapsulate things into `actors`
- Each actor represents one client or entity with local state and communicates via asynchronous messages
- `distributed actor frameworks` spreads the model to multi-node application
- Kind of like `RPC` because you send messages to multiple nodes, but `actor model` already assumes asynchronous messages and potential lost messages
- Example frameworks
  - `Akka`
  - `Orleans`
  - `Erlang OTP`
### Summary
- Rolling upgrades are good for no downtime deployments
- Puts a high requirement on forward and backward compatibility of data


# Part 2 - Distributed Data
- What happens if multiple machines are involved in storage and retrieval of data
  - Scalability
  - Fault tolerance
  - Latency
### Scaling to Higher Load
- `vertical scaling` or `shared-memory architecture` is simple, but cost-benefit is expensive
- `shared-disk architecture` - several machines with independent CPU and RAM but store data on an array of disk shared between machines
#### Shared-Nothing Architectures
- `shared-nothing architecture` or `horizontal scaling` is gaining popularity
- Each machine or `node` is independent
- All coordination is done at software level
- No special hardware requirements
- Requires a lot of caution by the application developer
#### Replication Versus Partitioning
- `Replication` - Keep copy of same data on multiple nodes
- `Partitioning` - Split a big database into smaller subsets
- We often use both partitioning and replication

## Chapter 5: Replication
- For now assume dataset can fit on each machine.  Later on consider partitioning
- Replication of static data is easy, the challenge comes from handling `changes` to replicated data
- Three popular algorithms
  - `single leader`
  - `multi-leader`
  - `leaderless replication`
### Leaders and Followers
- Most common solution is `leader-based replication` or `master-slave replication`
- One replica is designated `leader` and gets all writes
- Other replicas are `followers or secondaries`
- All writes to `leader` also then send data to followers as part of `replication log` or `change stream`
- Reads can come from leader or follower
- Common option in Postgres, MySQL, Oracle Data Guard, Mssql Always On Availability Groups
#### Synchronous Versus Asynchronous Replication
- Synchronous replication can be good as it guarantees all replicas are in sync
- But it is bad because if a follower doesn't respond, writes get blocked
- Usually if you have synchronous replication, only one follower is synchronous and others are asynchronous
  - Sometimes called `semi-synchronous`
#### Setting Up New Followers
- Take a snapshot of leader's database at some point, like a full backup
- copy snapshot to new follower node
- Follower connects to leader and asks for all data changes that happened after the snapshot
- Follower can use this data to catch up then process as normal as a new follower
#### Handling Node Outages
##### Follower failure: Catch-up recovery
- Follower can connect to leader and request all data changes since the disconnect then catchup
##### Leader failure: Failover
- Manual Failover vs Automatic Failover
- Automatic Failover
  - Determine that Leader Failed - difficult problem, usually rely on timeouts
  - Choose new leader - either election process or a leader chosen by previously elected `controller node`
  - Reconfigure system to use new leader
- Challenges in Failover
  - Asynchronous replication - new leader may not be fully up to date - usually throw them away and force application to deal with it
  - Dependency with External systems
    - e.g. Github had an outage where they promoted an out-of-date MySQL instance to be leader.  PKeys were autoincrementing but were behind, so re-assigned existing PKeys and wound up with duplicate mismatching entries compared to Redis.  Resulted in incorrect data being exposed
  - Must avoid split brain situtation
  - What is appropriate timeout value before failover? Hard question, shouldn't be too fast or too slow
#### Implementation of Replication Logs
##### Statement-based replication
- all statements get sent to all followers
- Challenges:
  - Some functions like now() or rand() will behave differently on other replicas
  - statements that depend on stuff like autoincrementing columns are easy to break
  - statements with side effects like triggers can cause different side effects on each replica
- Because of these challenges, usually other solutions are preferred
##### Write-ahead log (WAL) shipping
- Used by Postgres/Oracle
- Shortcoming: WAL replication ties storage format very closely to replication
  - This means you cannot have mismatching versions of the db software on source/target
  - Can remove the ability to do cold side upgrades first
##### Logical (row-based) log replication
- Separate the `logical log` from the physical data representation
- Transaction that modifies rows can generate records in the logical log
- Easier to keep compatibility even with differing versions
- Easier to send to external systems as well - `change data capture`
##### Trigger-based replication
- You can write a trigger to log a change into a dedicated table upon updates, which can be read by external systems
- More buggy and higher overhead than other methods, but more flexible
### Problems with Replication Lag
- Replication is helpful for high availability, localizing reads, etc
- In web world, convenient to scale up may read-only replicas for read distribution
- Can introduce lots of asynchronous problems
#### Reading Your Own Writes
- Challenge: If user submits data then reads from another replica, new data may not be in the replica yet, which is bad
- Options:
  - When reading something the user may have modified, read from leader
  - If too much stuff is potentially modified by user, this wont work.  We can try other mechanisms:
    - track time of last update and only read from leader for the first minute
    - track the latency to followers and stop queries to followers if latency is too high
  - Client remember timestamp of recent writes, make sure the replica is up to date to that time
  - Additional complexity if you have multi-dc: must also route to the appropriate datacenter for local reads
- What if user may access from multiple devices like web browser and mobile app
  - We want to provide `cross-device` read-after-write consistency
  - Much harder to "remember most recent write" and that needs to be centralized
  - Different devices may be on different network, so for multi-dc approach you may need to make sure you route requests to the proper DC
#### Monotonic Reads
- Bad scenario: What if user gets data from one follower, then gets data from a follower more out of date
  - This causes them to get older data on newer fetches
- `Monotonic reads` is a guarantee to avoid this situation
- Not as strong as `strong consistency` but stronger than `eventual consistency`
- One simple solution is to ensure each client always is routed to same follower
#### Consistent Prefix Reads
- `Consistent prefix reads` is a guarantee that if a series or sequence of writes happens in some order, then all reads of those writes will happen in the same order
- Big challenge in partitioned databases
- You can have some logic to track causal dependencies
#### Solutions for Replication Lag
- Don't pretend asynchronous replication is synchronous
- If lag is a problem, think about adding stronger promises like read-after-write
### Multi-Leader Replication
#### Use Cases for Multi-Leader Replication
##### Multi-datacenter operation
- One concept is to set up a leader in each data center, and each data center locally has followers
- Performance
  - Unlike single leader model, now writes can always go to a local datacenter
- Tolerance of datacenter outages
- Tolerance of network problems - Temporary network issues wont cause any writes from being processed
- New challenge: Write conflicts from multiple leaders
- Things like autoincrementing keys, triggers, and integrity constraints can cause issues, so usually we try to avoid for products that aren't designed for it
##### Clients with offline operation
- Offline app like calendar needs a local db that acts like a local leader
- Behavior is similar to a multi-leader setup
- Some tools like CouchDB are designed for multi-leader configuration
##### Collaborative editing
- Apps like google docs also introduce similar problem if multiple people edit the same doc
- Either implement a lock and only allow one editor at a time
- Or allow simultaneous editing but you must solve the conflict resolution
#### Handling Write Conflicts
##### Synchronous versus asynchronous conflict detection
- Synchronous conflict detection - wait for write to be replicated to all replicas before returning success - basically defeats the purpose of multi leader
- In multi-leader mode, usually we will have asynchronous conflict detection
##### Conflict avoidance
- Try to design the app to avoid conflicts
- For example route specific users to a specific leader and always use the same leader for that user to avoid conflicts
##### Converging toward a consistent state
- What if one replica processes A then B then C, but another one processes A then C then B.  Then two replicas will have different most recent value
- We must ensure we resolve conflicts in a `convergent` way
  - Give each write a unique ID, pick the write with highest ID as winner and throw away the older writes - `last write wins`
  - Give each replica a unique ID, and create a predefined order of precedence of which replica is most trusted
  - Create algorithm to merge values together
  - Record the conflict in a special data structure and create application code to solve it later, maybe with user confirmation
##### Custom conflict resolution logic
- Many multi-leader replication tools let you write your own custom code to deal with it
- `On write` - The special code gets triggered as soon as conflict is detected
- `On read` - When data is accessed, conflict resolution code is called.  May be automated or ask for user input
- Conflict resolution is usually at a row or document level, not transaction.  One transaction can have multiple conflict resolutions required
##### Automatic Conflict Resolution
- `Conflict-free replicated datatypes` aka `CRDT` - data structures for sets, maps, ordered lists, counters, etc that can be concurrently edited by multiple users
- `Mergeable persistent data structures` - track history explicitly like GIT and use a 3 way merge function
- `Operational transformation` - Conflict resolution algorithm used by things like Google Docs.  Designed for editing ordered list of items, such as the list of characters in a text document
##### What is a conflict?
- Some cases are hard to detect, like what if we have a room reservation system, and multiple reservations come into different leaders
#### Multi-Leader Replication Topologies
- Need to consider `replication topology`.  Some popular ones:
  - `circular topology` - each leader passes writes to the next one in a loop
  - `star topology`
  - `all-to-all` - most general topology
- With circular writes, you need to know how to stop an infinite loop when passing data round.  In the `replication log` make sure to tag the identifiers of which nodes each write has passed through
- `all-to-all` is reliable/resilient, but introduces problems related to causality
  - What if client A updates leader A first, then client B updates leader B second, but the update in leader B happens before replication from leader A arrives?
    - Use `version vectors` to try to solve this
### Leaderless Replication
- Amazon's `Dynamo`, `Riak`, `Cassandra`, `Voldemort` use leaderless replication
- Sometimes client can send writes to multiple replicas
- Other times a coordinator node does this for the client, but doesn't enforce ordering of writes
#### Writing to the Database When a Node Is Down
- Hypothetical: 3 node leaderless setup and one node is currently down
  - Client sends a write to all 3 replicas, 2 succeed and one fails
  - When the offline node comes back up, it has missed all writes while it was down
  - Clients will make sure to send `read requests` to multiple nodes and use version numbers of data to see which one is most up to date
##### Read repair and anti-entropy
- How can the offline replica catchup when its back?  `Dynamo` solution:
  - `Read repair` - Client reads from multiple replicas.  When it finds data out of sync, it can update back to the stale data with the new values.  Good for frequently accessed data
  - `Anti-entropy process` - Have a background process that looks for data inconsistencies and copies missing data from one replica to another
##### Quorums for reading and writing
- If we have `n` replicas, every write must be confirmed by `w` nodes and query at least `r` nodes for reads
- Rule is that `w + r > n` - This is called `quorum` reads and writes
- A common decision is to make `n` odd like 3 or 5 and set `w = r = (n + 1) / 2`
- With `n = 3` then `w = 2` and `r = 2` - We can tolerate one node down
- With `n = 5` then `w = 3` and `r = 3` - We can tolerate two nodes down
- If less than `w` or `r` nodes are available, you get an error
#### Limitations of Quorum Consistency
- You can try to set `w` and `r` smaller so that `w + r <= n`, and then you can be more tolerant of failures, but will start reading stale data
- Even with `w + r > n` there are edge cases:
  - `Sloppy quorum` - writes may end up on different nodes than reads, so you lose the guarantee to see most recent data
  - If two concurrent writes happen, its not clear which one is the winner
  - If read and write happen concurrently, the write may not have arrived at all replicas
  - If writes fail on some replicas due to problems, its not rolled back on the success replicas
  - If you need to restore a node from a replica with older data, you lose newer data
##### Monitoring staleness
- In leader based replication, database can expose metrics on latency
- In leaderless solutions it is much harder.  Especially if we use `read repair` only, data can be extremely stale
#### Sloppy Quorums and Hinted Handoff
- Scenario: network outage cuts off client from many nodes of database
  - Nodes are up, but client cannot reach them
  - Cannot achieve quorum due to insufficient nodes reached
  - Option 1: Fail all requests
  - Option 2: Accept new writes but maybe write them to nodes that aren't supposed to house this data: `sloppy quorum`
    - Once network is fixed, data is sent to "home" nodes - `hinted handoff`
##### Multi-datacenter operation
- Leaderless replication can also be good for multi-datacenter replication
- Maybe treat remote datacenters as asynchronous to avoid performance degradation
#### Detecting Concurrent Writes
- Background: If multiple clients are writing to multiple nodes simultaneously, each write may arrive at each node with different timing.  In that case, the subsequent read may be different depending on which write arrived first
##### Last write wins (discarding concurrent writes)
- Not perfect, as by definition we are talking about concurrent writes so there isn't really a last write
- We can attach a timestamp to each write and pick the biggest one as most recent
- You may then just "lose" data for the concurrent writes that don't win
- To avoid data loss, you can try to design data to always have a new key, to avoid conflicts
##### The "happens-before" relationship and concurrency
- New way of defining "concurrent" - don't look at the timing itself, just look at logical dependency
  - If update B depends on update A, then B is `causally dependent` on A
  - If update B and update A are unrelated, then we can call them `concurrent` (even if timing is totally different)
- We can boil this down to 3 possibilities:
  - A happened before B
  - B happened before A
  - A and B are concurrent
- Need an algorithm to check if A and B are concurrent
##### Capturing the happens-before relationship
- **P.188-189** - Walk-through of situation where 2 clients keep updating the same key
- Use data `versions` and let the client handle conflicts when it receives multiple versions back from server
- Algorithm overview:
  - Server keeps version number for each key, increments it each time key is written
  - When client reads a key, server returns all values that have not been overwritten and the latest version number
  - When client writes a key, it must tell server which version number its using from previous read AND merge together all values from previous read
  - When server receives a write with a version number, it can overwrite the values from that number or earlier, but must keep any values with a higher version number (these are concurrent writes)
##### Merging concurrently written values
- Client now has extra work to merge conflicts
- Similar to the conflict resolution for multi-leader replication
- Options
  - Take the most recent version - loses data
  - Just *union* all the data together - but now we cannot properly handle deletes happening in each version
    - Can use a `tombstone` in our data to mark that data is being deleted
- Merging siblings is complex, data structures like CRDTs try to help with this
##### Version vectors
- How does this algorithm change if there are *multiple* replicas, not just one copy?
- Now *each* replica must keep a version number of data per key - the collection of version numbers from all replicas is called `version vector`
- Before each write, first always read data and get the full version vector
- Then make sure to resolve any conflicts and write back
- Safe to read from one replica and write to another replica
### Summary
- Replication can be useful for:
  - High availability
  - Disconnected operation during network issues
  - Latency/localization of data
  - Scalability
- 3 main approaches to replication
  - Single leader
  - Multi leader
  - Leaderless
- Replication lag scenarios and consistency models
  - Read-after-write consistency
  - Monotonic reads
  - Consistent prefix reads

## Chapter 6: Partitioning
- Use a `shared nothing` architecture to shard data and distribute across many disks
- Look at approaches for partitioning, see how indexing interacts with partitioning
- Then talk about rebalancing
- How to route requests to the correct partition
### Partitioning and Replication
- Often we want both partitioning and replication
- We can have separate partition concept from node concept
- Then have layout like:
  - Node 1 has partition 1 leader, partition 2 follower, partition 3 follower
  - Node 2 has Partition 2 follower, Partition 3 leader, partition 4 follower
  - etc
### Partitioning of Key-Value Data
- We want to have even partitioning
- Avoid `skew` or `hot spot` partitions
- Random/round robin distribution will avoid skew, but then reads are harder
#### Partitioning by Key Range
- Similar to an encyclopedia concept, each partition can be responsible for a range of values
- Partitioning can be done manually or automatically
- Within each partition we can keep keys sorted to make range scans easy
- Be careful with the type of data though
  - Example: Reading sensor data and partitioning by the timestamp of measurement
  - All writes will always go to the exact same partition on each day
  - Maybe need to have a different first element of the key, such as sensor name
#### Partitioning by Hash of Key
- Find a hash function that will output good distribution of values
- Then we can partition by the range of the hashes
- Boundaries can be evenly spaced or chosen pseudorandomly (`consistent hashing`)
- Now we lose the range query benefit from range partitioning
  - Cassandra tries to offer compromise where `compound primary key` can use first column for hashing, then the other columns can be sorted to allow for range queries
#### Skewed Workloads and Relieving Hot Spots
- Hash partitioning helps avoid hot spots, but not 100%
- Example: social media site where one celebrity has millions of followers, so the same key may become a hot spot
  - Application must handle any special logic to reduce hot spot
  - Example: Add a 2 digit random number to the unique key so that it can be distributed across 100 keys
    - Downside is that now reads must re-aggregate the 100 keys in order to read the activity for that unique ID
### Partitioning and Secondary Indexes
#### Partitioning Secondary Indexes by Document
- Example: website for selling used cars
  - Unique ID, document ID can be used for partitioning by range
  - Then we add secondary indexes for *color* and *make*
  - Each partition can then store its own secondary indexes for each color and each make
  - Within any given partition, we can easily search for all red cars or all Ford cars
  - This is called `local index`  
  - Downside is that any global reads based on color must check the local index on all partitions - `scatter/gather` approach can be expensive
  - Vendors recommend to try to design queries on these indexes to stay within one partition
#### Partitioning Secondary Indexes by Term
- As opposed to `local index` we can create a `global index` - Cover the data of all partitions in a single place
- However we dont want to just place the index on one node, so then we will also partition our `global index` across all nodes, based on the `term`
  - Like our data, we can decide to partition our `global index` by range or hash of the term
- Reads will be improved because we don't have to do scatter gather anymore
- But writes are harder because we may need to update multiple terms of global index on multiple partitions for each write
- In practice, many global indexes have asynchronous write behavior
### Rebalancing Partitions
- Key requirements
  - after rebalancing, load should be fairly distributed
  - During rebalancing, db should be able to still do reads and writes
  - Try to minimize data movement to avoid flooding network and disk I/O
#### Strategies for Rebalancing
##### How not to do it: hash mod N
- Simple naive approach is to take mod of the hash then use that to assign the partition
- Problem is that during rebalancing, all the mod values will change and pretty much all data needs to move
##### Fixed number of partitions
- Create some large number of partitions up front, much higher than the number of nodes
  - e.g. Create 1000 partitions for 10 nodes, and put 100 partitions on each node
  - If you need to add nodes, just move partitions in an even amount to the new node(s)
##### Dynamic partitioning
- When a partition grows to a certain size, the db can split it into two partitions of ~equal size
- If data is deleted and a partition shrinks, it can be merged into a neighbor partition
- After partition split, one of the halves can move to another node 
- Usually you start with only one partition until data grows enough to start splitting, so other nodes start out idle
- Some enhancements allow for some up-front splitting depending on the product
##### Partitioning proportionally to nodes
- dynamic partitions - number of partitions are proportional to data set size
- fixed partitions - size of partitions is proportional to the size of data set
- Try to make the number of partitions proportional to the number of nodes - fixed number of partitions per node
- When new node joins cluster, choose a number of partitions to split and take ownership of half of each of the split ones
  - some splits are uneven, but when we have 256 partitions per node, it averages out
#### Operations: Automatic or Manual Rebalancing
- Fully automated rebalancing can be dangerous
  - Automatic failure detection plus automatic rebalancing could happen if one node is slow and cause major overhead
- Doesn't have to be fully manual, can have automated proposal with human operator confirmation
### Request Routing
- Service Discovery is a challenge
- Common approaches
  - Client connects to any node, if data is on another node, that node can forward the request and then pass result back to client
  - Create a routing tier first, which finds data and reroutes to the appropriate node
  - Make the client aware of partitioning and have it directly connect to the proper node
- How can the routing mechanism be aware of rebalancing?
- Often we use an external coordination service like ZooKeeper to track the cluster metadata
- Another approach is to use a `gossip protocol` to spread information about changes in cluster state
#### Parallel Query Execution
- Read/write single key is easy
- `Massively parallel processing` db products have much more difficult problem
  - MPP query optimizer breaks query into execution stages and partitions and tries to execute in parallel
### Summary
- Two main approaches to partitioning
  - Key range partitioning
  - Hash partitioning
- Interaction of partitioning and secondary indexes
  - Document partitioned indexes/local indexes
  - Term-partitioned indexes/global indexes

## Chapter 7: Transactions
- Transactions have been a main mechanism for simplifying potential problems
- Group several reads and writes into one logical unit
  - Can either commit or else abort or rollback
- Need to understand what safety we get from transactions, and the cost
### The Slippery Concept of a Transaction
- NoSQL dbs offered replication and partitioning by default, but often started to reduce the transaction support
- NoSQL world started to have view that transactions make performance impossible
- On the other hand, database vendors present transactions as requirement for serious data
- Need a more nuanced view
#### The Meaning of ACID
- ACID can have varying interpretations
##### Atomicity
##### Consistency
- In ACID context, consistency means the database is in a "good state" from application perspective
- `Invariants` (aka statements) about your data must be true
  - e.g. all credits and debits must be balanced
- Consistency can only be controlled by the app, not by the db...doesn't really belong in ACID
##### Isolation
- concurrently executing transactions are isolated from each other
- Classic textbooks refer to this as `serializability` - database ensures that transactions behave as if they had run serially
  - In practice this is not used or implemented often
##### Durability
- Once data is written, it wont be forgotten
- Write data to disk
- Also use a write-ahead log
#### Single-Object and Multi-Object Operations
##### Single-object writes
- How would db respond if something goes wrong in the middle of writing a 20KB JSON Document
- Almost universally all engines try to provide atomicity and isolation
  - Atomicity is done using a log for crash recovery
  - isolation via locks
- Some dbs provide other operations like increment operation or compare-and-set operation - only does update if not being concurrently changed by someone else
- These can be seen as "lightweight transactions"
##### The need for multi-object transactions
- Many distributed datastores don't have multi-object transactions because they are difficult across partitions 
- This can be a challenge as many scenarios need a transaction that can cover multiple objects
  - Without transactions, error handling in the application is much more difficult
##### Handling errors and aborts
- Relational DBs and ACID data stores: Prefer to abort a transaction rather than allow half-transactions
- Leaderless replication systems often are more of a best effort basis
- Usually if a transaction fails or aborts, simply retrying is a good approach, but there are some edge cases
  - transaction actually succeeded but network failed so we double apply in this case
  - error due to performance is exacerbated by retry logic
  - Side effects outside of the database could be undesirable, such as sending an email each time
### Weak Isolation Levels
- Weak isolation can lead to real world bugs and problems
- Not so easy to "just use ACID database" because even popular RDBMS can use weak isolation
#### Read Committed
- When reading from database, you see only committed data (no dirty reads)
- When writing to database, you only overwrite committed data (no dirty writes)
##### No dirty reads
- Useful to make sure you don't see "half of a transaction" for other processes reading data while a process is updating it
- Prevents other apps from seeing data that would later get rolled back
##### No dirty writes
- Dirty writes can prevent race conditions when multiple users are trying to update multiple objects
  - e.g. buying a car on a website with 2 updates: set the buyer, and then set the invoice
  - In a scenario where Alice gets update for buyer first and Bob gets buyer second, but sets invoice first, then Bob would be last write wins for buyer, but not for invoice
  - Then Bob becomes buyer but Alice gets invoice so its wrong
- Read Committed (aka no dirty writes) can NOT prevent race condition when doing an update of a counter, as both writes will update the counter and it will just get incremented more times than we wanted
##### Implementing read committed
- Read committed is a popular isolation level and default in many RDBMS
- Usually we use row-level locks to achieve this
- Many databases then make sure to save both the old value and new value of a row while a lock is held
  - Then other processes that want to read while its being written to can get the old value 
  - Only once the write is complete and the lock is released would other processes see the new value
#### Snapshot Isolation and Repeatable Read
- `Nonrepeatable read` or `read skew` scenario
  - Someone is doing bank transfer from account A to account B
  - Alice reads the balance in accountA before transfer, but reads balance in accountB after transfer
  - Alice now has read data both before and after the transaction and is not seeing the correct total value across the two accounts
- This scenario does NOT violate `read committed` expectations
- Scenarios that cannot accept nonrepeatable reads:
  - Backup of whole database
  - Analytics queries and integrity checks
- `Snapshot isolation` is common solution to this problem
  - Each transaction reads from a `consistent snapshot` of the db, ie the version of the data as of the start of the transaction
##### Implementing snapshot isolation
- Key principle is that readers never block writers and writes never block readers
  - Database can have long running reads and also process writes at the same time
- Multi Version Concurrency Control (MVCC) is used
- Postgres implementation example
  - Each transaction gets a unique increasing transaction id aka txid
  - All data changes are tagged with the txid
  - Internally each row has a created_by and deleted_by field
  - Updates are managed by doing a deleted_by and then created_by with new value
##### Visibility rules for observing a consistent snapshot
- When transaction starts, database checks all other in-progress transactions.  Writes by those transactions are ignored
- Writes by aborted transactions are ignored
- Writes by transactions with txid higher than this one are ignored
- All other writes are visible to the application's queries
##### Indexes and snapshot isolation
- How does multi-version affect db? should we index all versions of the object?
- Some dbs like CouchDB/Datomic/LMDB use B-trees but with a `append-only/copy-on-write` variation
  - Pages are not changed/overwritten but instead new copy is created
  - Parent pages up tot the root are copied and updated to point to new version of child pages
  - Append-only B-trees - each write transaction creates a new B-tree root that is consistent to the snapshot of the db at the time when it was created
  - This way we dont have to store each version inside the b-tree
##### Repeatable read and naming confusion
- Diffent products will call snapshot isolation by various names
  - `serializable` in oracle
  - `repeatable read` in Postgres/mysql
- SQL standard from 1975 didn't have snapshot isolation and the official isolation levels are ambiguous
#### Preventing Lost Updates
- Another difficult problem besides dirty writes is two concurrent writes
- `Lost Update` problem
  - Two different processes read a value, modify the value, then write it back, such as a counter increment
  - Occurs when:
    - incrementing a counter or updating an account balance
    - Making a local change to a complex structure like a JSON document
    - multiple users editing a wiki page
##### Atomic write operations
- Try to use options like `Update counters et value = value + 1 where key = 'foo'`
- Document dbs like MongoDB provide atomic operations
- Some use-cases like wikipedia updates can't really support that
- Be careful of abstractions like ORMs that can hide the fact that something is not done atomically
##### Explicit Locking
- Application can explicitly lock objects that will be updated
- Be careful in app code to avoid race conditions
##### Automatically detecting lost updates
- Allow parallel execution, and if transaction manager detects a lost update, abort transaction and force a retry of read-modify-write cycle
- Many dbs offer support for his via snapshot isolation/etc
- MySQL does NOT support this, so this isn't an option with all db products
##### Compare-and-set
- Allows an update only if the value hasn't changed since you read it
  - If value changed, no update happens so you must read-modify-write again
##### Conflict resolution and replication
- Locks and compare-and-set operations assume that there is one single copy of data
- Common approach is to allow concurrent writes to create conflicting versions aka siblings of data and use code to resolve conflicts later
- Atomic or `commutative` (can apply in any order) changes work well in replication
- `Last-write-wins` conflict resolution often generates lost updates
#### Write Skew and Phantoms
- Example case: Hospital requires at least one doctor on shift.  Both Alice and Bob are on shift and need to cancel their shift
  - Both of them do begin tran; check count of current oncall doctors; update oncall set on_call=false for themself
  - Due to snapshot isolation, both of them read the value of 2 doctors on call, but now their application logic allows both of them to set false, and now there are 0 doctors on call
##### Characterizing write skew
- Above scenario is called `write skew` and is different from dirty writes and lost updates because they are NOT updating the same object
- Write skew is harder to solve than lost update problem because we are enforcing application requirements across multiple objects
- One approach that can work is to explicitly lock rows of the transaction to prevent multiple updates from happening at the same time
##### More examples of write skew
- Meeting booking room system
  - Check if any meetings affect a room.  If not, then create a booking in that room
  - Snapshot isolation cannot prevent double booking if simultaneous
  - You need `serializable` isolation
- Multiplayer game
  - How to prevent two players from moving different characters to the same position on a board at the same time
- Claiming a Username
  - Check if a username exists and create it atomically - still not safe with snapshot isolation
  - Unique key constraints can solve this easily
- Preventing double-spending
  - A service that lets users spend money or points needs to enforce that user doesn't spend more than they have
  - Maybe you check this by checking the sum of price for a transaction, put it in user's account and make sure sum is still positive
    - If you do this twice simultaneously, both charges can return sufficient funds, but both together are over the limit
##### Phantoms causing write skew
- General pattern of this issue
  - SELECT query checks some condition
  - Based on select results, application decides what to do
  - If application goes ahead, it will update data in db
  - The result of the update of data changes the result of the original SELECT query
- If the update affects the SAME object as the original SELECT query, its a bit easier
- But if the update will touch another object different from the original SELECT query, or checks for *absence* of rows, it is harder
- Write in one transaction changes result of search query in another transaction is called a `phantom`
##### Materializing conflicts
- One solution is to create some kind of object specifically to have locks on whatever you need to resolve conflict on
- For example with room booking app you can create a room-timeslot-lock table with a slot for every 15 minutes
  - Now app can lock (select for update) the corresponding rows in the room-timeslot-lock table to ensure there are no conflicts
- This approach is called `materializing conflicts`
- This approach works but is considered ugly and error-prone, so use as a last resort
- Serializable isolation level is preferable in most cases
### Serializability
- Serializable isolation can solve a lot of our concurrency issues
- Guarantees that even transactions that happen in parallel happen as if they were done serially
- Need to look at the methods for achieving serializability and how they perform
  - Literally execute things serially
  - Two-phase locking
  - Optimistic concurrency control techniques
#### Actual Serial Execution
- Literally only have one thread that can make changes
- Only around 2007 did this start to become feasible
  - RAM is cheap enough to keep active data in memory
  - OLTP transactions are short enough and make small enough reads/writes
- This is used in VoltDB, Redis, and Datomic
- Performance can improve from lack of locks/coordination, but limited by one core
##### Encapsulating transactions in stored procedures
- Serial execution system needs to ensure that interactive multi-statement transactions dont happen or it would kill performance
- You must send the entire transaction to the database at once, as a stored procedure
##### Pros and cons of stored procedures
- Stored procs have a bad reputation due to
  - different flavors of sql in each product
  - code in db is much harder to maintain
  - database is performance sensitive compared to app server so bad stored procs can really cause peformance issues
- Modern stored procs move away from SQL toward languages like java/groovy/lua
- Stored procs can execute fast enough when data is in memory
##### Partitioning
- To get around the write bottleneck of one CPU, you can partition data
- If transaction needs to access multiple partitions, you need to coordinate across the partitions
##### Summary of serial execution
- Each transaction needs to be small and fast
- Limited to cases where data can fit in memory
- Write throughput must be low enough to be handled on one cpu
- Cross-partition transactions are possible but slow
#### Two-Phase Locking (2PL)
- Two phase lock is different from Two phase commit
- Two phase lock was the only widely used algorithm for serializability for 30 years
- Rule is:
  - Multiple transactions can read concurrently from object
  - As soon as anyone wants to write the object, exclusive access is required
    - All transactions that want to write to an object must wait until current reads are completed
    - Any transaction that wants to read now must wait for the write lock to complete
    - Aka writers block writers AND readers
    - Readers block writers
    - In snapshot isolation, we have readers never block writers and writers never block readers
##### Implementation of two-phase locking
- Locks can be in shared mode or exclusive mode
- To read an object, you need a shared lock. Simultaneous shared locks are allowed, but not if an exclusive lock is already there
- To write an object you need a exclusive lock.  No other locks are allowed while exclusive lock is there
- If a transaction reads then wants to write to an object, you can upgrade lock to exclusive lock
- After transaction gets a lock, it is not released until transaction is committed or aborted
- Deadlocks are pretty easy to generate in this model
##### Performance of two-phase locking
- Two phase locking guarantees reduced concurrency and therefore unstable performance
- One transaction can halt everything
##### Predicate locks
- `predicate lock` - works similar to shared/exclusive locks but instead of lock on an object, it belongs to ALL objects that match some criteria
- e.g. we can have a predicate lock on this condition:
```
SELECT * from bookings
  WHERE room_id = 123 AND 
  end_time > '2018-01-01 12:00' AND
  start_time < '2018-01-01 13:00';
```
- Transaction can get shared mode predicate lock for reading or exclusive lock for modifying anything that matches these conditions
- When doing update/insert/delete, first check if the new OR old value would match a predicate condition, and if so then get the exclusive predicate lock
##### Index-range locks
- Predicate locks dont have good performance so many databases use `index-range locking` which is a simpler version
- You lazily get a lock on some other condition that will affect *more* rows but is easier to get a lock on
  - e.g. instead of locking bookings on room123 from noon to 1pm, you can lock all bookings on room 123, or lock all bookings between noon and 1pm
- You would have an index on either room_id or the booking_time
  - database attaches shared lock to the relevant index or a range of values on that index
- Overall this is an approximation of the predicate lock; less precise but better performance
#### Serializable Snapshot Isolation (SSI)
- SSI was first described in 2008 and provides full serializability but less performance penalty compared to snapshot isolation
- SSI is used by Postgres in serializable isolation level and in FoundationDB
##### Pessimistic versus optimistic concurrency control
- Two phase locking is `pessimistic` control mechanism; better to wait until everything is safe before doing anything
- `Serializable Snapshot Isolation` is `optimistic` concurrency control technique
  - Instead of blocking dangerous things, we go ahead and do it, then at commit time check if any bad things happened
  - Can perform badly on high contention systems that will generate lots of aborts
  - If there is some leeway in capacity and system is not overloaded overall, then optimistic algorithm can perform better than pessimistic
- Overall this is based on snapshot isolation, and then on top of this it adds an algorithm to detect serialization conflicts so it knows when to abort a transaction
##### Decisions based on an outdated premise
- Two scenarios the db must detect to make sure the transaction is not being committed based on a `stale premise`
  - Detect reads from a stale MVCC object version 
  - Detect writes that affect prior reads
##### Detecting stale MVCC reads
- snapshot isolation is usually achieved via MVCC
- Example scenario:
  - Doctor oncall booking system
  - Alice queries oncall doctors and then updates self to not be oncall
  - While this transaction is open and before Alice's write is done, new transaction starts checking who is oncall then updates Bob to be not oncall
  - At commit time, second transaction will detect that the read was based on stale MVCC data and abort
  - But this optimistic method helps because we didn't yet know if Alice's transaction would update data or maybe fail to commit
##### Detecting writes that affect prior reads
- Kind of similar idea to `index range locks`, but we want to use our lock as a "tripwire" mechanism
- The database will attach data to an index (or base table if no index) about which transactions have read this data recently
- This data can be cleared out once a transaction finishes and/or concurrent transactions are done
- When a transaction wants to do a write on the db, it searches indexes for any other transactions that recently read this data
- Whichever transaction commits first will be fine, but whoever commits second knows the "tripwire" was tripped and must fail
##### Performance of serializable snapshot isolation
- Lots of engineering details/performance enhancements can affect overall performance
- Main advantage vs 2PL is you don't need to wait for locks
- Long running transactions tend to cause issued and don't perform well with SSI
### Summary
- Transactions are a useful abstraction to allow applications to pretend some concurrency issues etc don't exist
- Discussed some widely used isolation levels like `read committed`, `snapshot isolation` aka `repeatable read` and `serializable`
- Discussed some race conditions
  - `dirty reads` - one client reads another client's writes before its committed
  - `dirty writes` - one client overwrites data another client has written but not yet committed (almost always prevented by transactions)
  - `read skew` - client sees different parts of db at different points in time.  usually prevented via snapshot isolation/mvcc
  - `lost updates` - 2 clients simultaneously do a read-modify-write cycle and override the value without considering the other's changes
  - `write skew` - transaction reads data, makes decision, then updates data, but by the time the write happens, the premise is no longer true
  - `phantom reads` - transaction reads objects that match some condition, but another client makes a write that affects the result of that query.  can be solved by special things like index-range locks
- Weak isolation levels protects some things but puts the rest of the onus on the developer
- Serializable isolation is achieved by 3 possible options
  - Literally execute serially
  - Two phase locking
  - Serializable snapshot isolation

## Chapter 8: The Trouble with Distributed Systems
### Faults and Partial Failures
#### Cloud Computing and Supercomputing
### Unreliable Networks
#### Network Faults in Practice
#### Detecting Faults
#### Timeouts and Unbounded Delays
#### Synchronous Versus Asynchronous Networks
### Unreliable Clocks
#### Monotonic Versus Time-of-Day Clocks
#### Clock Synchronization and Accuracy
#### Relying on Synchronized Clocks
#### Process Pauses
### Knowledge, Truth, and Lies
#### The Truth Is Defined by the Majority
#### Byzantine Faults
#### System Model and Reality
### Summary

## Chapter 9: Consistency and Consensus


# Part 3 - Derived Data
## Chapter 10: Batch Processing
## Chapter 11: Stream Processing
## Chapter 12: The Future of Data Systems