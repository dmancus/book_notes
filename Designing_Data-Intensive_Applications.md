# Designing Data-Intensive Applications

# Table Of Contents
  - [Part 1 - Foundations of Data Systems ](#part-1---foundations-of-data-systems)
  - [Part 2 - Distributed Data](#part-2---distributed-data)


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
#### Synchronous Versus Asynchronous Replication
#### Setting Up New Followers
#### Handling Node Outages
#### Implementation of Replication Logs
### Problems with Replication Lag
#### Reading Your Own Writes
#### Monotonic Reads
#### Consistent Prefix Reads
#### Solutions for Replication Lag
### Multi-Leader Replication
#### Use Cases for Multi-Leader Replication
#### Handling Write Conflicts
#### Multi-Leader Replication Topologies
### Leaderless Replication
#### Writing to the Database When a Node Is Down
#### Limitations of Quorum Consistency
#### Sloppy Quorums and Hinted Handoff
#### Detecting Concurrent Writes
### Summary

## Chapter 6: Partitioning
## Chapter 7: Transactions
## Chapter 8: The Trouble with Distributed Systems
## Chapter 9: Consistency and Consensus


# Part 3 - Derived Data
## Chapter 10: Batch Processing
## Chapter 11: Stream Processing
## Chapter 12: The Future of Data Systems