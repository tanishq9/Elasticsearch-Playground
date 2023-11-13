## Introduction to ES

### Document in ES
- In ES, the data is stored as documents.
- A document in ES is similar to row in relational database (like MySQL) and can represent a person, sale, etc.
- A document consists of fields which correspond to column in relational database.
- A document is essentially a JSON object.

### Querying
- The way we query documents is to use a REST API.
- The queries sent to ES are also written in JSON.
- ES is written in Java and built on top of Apache Lucene.

### Overview of ELK
#### Kibana
- It is an analytics and visualisation platform, which lets you easily visualise data from ES, and analyze it to make sense of it. We can think of Kibana as ES dashboard where you can create visualisations such as pie charts, line charts, etc.
- Kibana is a web interface to the data that is stored within ES. It send queries using the same REST APIs.

#### Logstash
- Logstash can be used to process logs from applications and send them to ES.
- Logstash is a data processing pipeline, the data that Logstash receives, will be handled as events, which can be anything like log file entries, e-commerce orders, customers, chat messages, etc.
- These events are then processed by Logstash and shipped off to one or more destinations like ES, Kafka, HTTP endpoint, etc.
- Logstash pipeline consists of 3 parts or stages - inputs, filters, and outputs.
- There are a lot of input plugins like Kafka, file, HTTP, etc. that Logstash uses to receive events. Filter plugins are about how Logstash should process them, here in we can parse CSV, XML or JSON, and do data enrichment. Output plugins are where we send the processed events to, these places are also called stashes.

#### X-Pack
- X-Pack is pack of features that adds additional functionality to ES and Kibana.
- Security: X-Pack adds both AuthN and AuthZ to both ES and KB. We can control permissions using fine-grained authorization.
- Monitoring: Gain insight into how the Elastic Stack is running. Eg. CPU, memory usage, disk space, etc.
- Alerting: If a web server's CPU usage exceeds some x% then alert notification can be sent on slack/email channels.
- Reporting: We can export Kibana visualisations and dashboards to pdf files, reports can be generated on-demand or scheduled.
- Machine Learning: Enables Kibana to do ML. Anomaly detection, forecasting, etc.
- Graph: Show related products or songs.
  - The uncommonly common signals relevance.
  - Graph uses relevance capability of ES when determining what is related and what isn't.
  - Graph exposes an API, which can be used in applications.
  - It works out-of-the-box, so we don't need to index data in a certain way to use it.
- ES SQL: Query ES using SQL. ES queries are written in Query DSL.

#### Beats
- Beats is a collection of data shippers, we install them on servers and they send data to Logstash or ES.
- There are a number of data shippers called beats - that collect different kinds of data and serve different purposes.
- For example, there is a beat named Filebeat, which is used for collecting log files, and sending the log entries off to either Logstash or ES.

#### Summary
- Ingesting data into ES can be done with Beats and/or Logstash, but also directly through ES API.
- Kibana is a user interface that sits on top of ES and lets you visualise the data it retrieves from ES through the API.
- ELK stack: ES, Logstash and Kibana.

### Getting Started
- To start elasticsearch, and do various installation tasks. A super user would be created with password when above command was executed. Anytime, we communicate with elasticsearch, we need to provide these credentials.
```
bin/elasticsearch
```
- To reset the password: bin/elasticsearch-reset-password -u elastic
- An enrolment token is also provided which is required to communicate with Kibana, this token is valid for 30 minutes and can be regenerated with this command: bin/elasticsearch-create-enrollment-token --scope kibana

### Understanding the basic architecture
- Node is an instance of ES that stores data. Each node belongs to a cluster.
- To ensure we can store many TBs of data if we need to, we can run as many nodes as we want. Each node will then store a part of our data.
- Node refers to an instance of ES and NOT a machine, so you can run any number of nodes on the same machine. This means that on your development machine, you can start up five nodes if you want to.
- A cluster is a collection of related nodes that together contain all the data.

### Index Introduction
- Every document within ES is stored within an index.
- An index groups documents logically as well as provide configuration options that are related to scalability and availability.
- When we search for data, we specify the index, the search queries are run against indices.

### Sending queries via cURL
- cURL used via Postman for http request to ES:
```
curl --location 'https://localhost:9200/_cluster/health' \
--header 'Authorization: Basic ZWxhc3RpYzpYU2hKSUNfTlJRaW1lVFlNRGk5cQ=='
```
- Here, 
  - _cluster is the API.
  - health is the command.
  - Route convention: /api/command

- To generate the Authorization header, the following was done:
```
echo -n "elastic:XShJIC_NRQimeTYMDi9q" | base64
```
- https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/Getting%20Started/sending-queries-with-curl.md

### Sharing and Scalability
- Sharding is a way to divide indices into smaller pieces.
- Each piece is referred to as a shard.
- Sharding is done at index level.
- Shard is stored on a node.
  - Just to be clear, a shard may be placed on any node, so if an index has 5 shards, we don't need to spread these 5 shards on 5 different nodes, we could but we don't have to.
- The main purpose is to horizontally scale the data volume.

Shard size:
- A shard has no predefined size; it grows as documents are added to it.
- A shard may store upto about 2B documents.

Reason of sharding:
- Mainly to store more documents for a single index.
- Divide index into smaller chunks which could be stored on multiple nodes.
- Improved performance:
  - Search query can be run in parallel on multiple shards at same time.
  - Parallelisation of a query increases throughout of the index.

Note:
- 'pri' stands for number of primary shards.
- An index is divided into one or more shards, where each shard stores a part of the index data.
- An index will contain one shard by default for Elasticsearch >= 7.0.0.

#### Configuring the number of shards
- An index contains a single shard by default.
- Indices in ES > 7v were created with 5 shards.
  - This often led over sharding.
- To increase the number of shards, we can use Split API.
- To reduce the number of shards, we can Shrink API.

#### How many shards to choose?
- There are many factors involved, so it depends.
- Factors include the # of nodes and their capacity, the # of indices and their sizes, the # of queries, etc.
- If we are antiquating millions of documents for an index, then we can choose a few shards to begin with.

### Replication
- What happens if a node's hard drive fails?
  - Hardware can fail at any time, so we need to handle that somehow.
- ES supports replication for fault tolerance.

How replication works?
- Replication is configured at the index level, similar to sharding.
- Replication works by creating copies of shards, referred to as replica shards.
- A shard that has been replicated, is called a primary shard.
- A primary shard and its replica shards are referred to as a replication group.
- Replica shards are a complete copy of a shard.
- A replica shard can serve search requests, exactly like its primary shard.
- The number of replicas can be configured at index creation.

Important:
- Replica shards are never stored on the same node as their primary shard. This means that if one node disappears, there will always be at least one copy of a shard's data available on a different node.
- Replication only makes sense for clusters that contain more than one node, because otherwise replication is not going to help if the only available node breaks down.

Choosing the number of replica shards:
- How many replica shards are ideal, depends on the usecase.
- Eg: Is the data stored elsewhere, such as in RDBMS?
- Is it OK for data to be unavailable while you restore it?
- For mission critical systems, downtime is not acceptable.
- Replicate shards once if data loss is not a disaster.
- For critical systems, data should be replicated at least twice.

### Snapshots
- ES supports taking snapshots as backups.
- Snapshots can be used to restore to a given point in time.
- Snapshots can be taken at the index level, or for the entire cluster.
- Use snapshots for backups, and replication for high availability (and performance).
- The snapshot taken can be used to restore the state of the cluster or indices to that state.

Note:
- Incase we have 2 nodes, wherein one node has primary shard for a particular index and other node has replica shard for this index.
- If we increase the number of replica shards then that although not contribute to increasing availability of the index (since the index is still present in 2 nodes) BUT that would help to increase the throughput of the index as now the search query would be distributed across 3 shards.
- Therefore, replication would help to improve availability of index and also the throughput.
- Replication ensures that if a node goes down, data will be available on a different node (provided that the cluster consists of more than one node).

### Adding more nodes to cluster
- To add a node, similar we did for Kibana, we need an enrolment token. This token is only needed when starting up a new node for the first time.
- Run below command in existing node:
```
bin/elasticsearch-create-enrollment-token --scope node 
```
- To start other nodes in same cluster as above node:
```
bin/elasticsearch --enrollment-token <TOKEN>
```

### Node Roles
- Master
  - This node is responsible for performing cluster-wide operations, this includes creating and deleting indices, keeping track of nodes, and allocating shards to nodes.
  - Configuration: node.master: true | false
  - A node with this role will not automatically become the master node, it is decided on basis of voting.
    - Unless there are no other master-eligible nodes.
  - For large clusters, we can have dedicated master nodes, this is to ensure master node itself is not overloaded with other operations like searching which are expensive in terms of hardware resources. So if we see high cpu, memory and i/o usage on master node then it might be time to add a dedicated master node.
    - Having a stable master node is crucial for ensuring that the cluster is stable.
- Data
  - This enables a node to store data.
  - Storing data includes performing queries related to that data, such as search queries.
  - Configuration: node.data: true | false
    - This can be set as false for dedicated master node.
- Ingest
  - This enables a node to run ingest pipelines.
  - Ingest pipelines are series of steps run when indexing documents
    - Processors may manipulate documents, e.g. resolving an IP to latitude/longitude
  - This is a simplified version of Logstash, directly within ES.
  - Ingest pipelines are useful for simple data transformations.
  - If we run a lot of documents through these pipelines, you might consider having a dedicated node for this.
  - Configuration: node.ingest: true | false

Note:
- If there are 2 primary shards and 2 replication shards for an index, then this means 2 primary shards would be created for this index and for each of the primary shard, there would be 2 replica shards, i.e. total 1+2 + 1+2 = 6 shards (2 primary and 2 replica for each).

### Updating Documents
- ES documents are immutable.
- We actually replaced the documents using update api.
- The existing document is not updated infact it is replaced with the new modified document.

### Routing
How does ES know where to store documents?
How are documents found once they have been indexed?

The answer is routing. Routing is the process of determining shard for a document.

Routing Strategy:
- shard_num = hash(_routing) % num_primary_shards
  - _routing is the document id
- Searching for documents on basis of criteria other than IDs i.e. document_id works differently, this will be covered later.

#### How does ES read data?
- Request is sent to the coordinating node (this node role is responsible for determining the node which contains the requested data).
- The below formula is used: replication_group = hash(_routing) % num_primary_shards
  - The replication group is determined using document_id, the replication group consists of primary and replica shards information.
  - This ES node then chooses the best node (best in terms of performance) in the replication group to respond to this query. The algorithm used is called Adaptive Replica Selection (ARS).
  - Now the request goes to the shard (which could be in any node) to retrieve the requested data.

#### How ES writes data?
- Write operations are sent to primary shards by the coordinating node.
- The primary shard forwards the operation to its replica shards.
- Primary terms and sequence numbers are used to recover from failures.
  - Primary term is a counter which is incremented each time the primary shard changes.
  - Sequence number is a counter which is incremented for each write operation, by the primary shard. This enables ES to order the write operations.
  - Primary terms and sequence numbers are available within responses.
- Global and local checkpoints help speed up the recovery process.
  - These checkpoints are the sequence numbers.
  - Each replication group has a global checkpoint.
  - Each replica shard has a local checkpoint.
  - Global checkpoint
    - Sequence number that all active shards within a replication group have been aligned to.
  - Local checkpoint
    - The sequence number for the last write operation that was performed.
  - Using these 2 checkpoints, only the operations that have a sequence number higher than the local checkpoint need to be applied when a node comes back, therefore ES only needs to compare the operations while the shard was "gone" instead the entire history of the replication group.

### Document Versioning
- Versioning is not a revision history of a document.
- ES only stores the most recent version of a document.
- ES stores _version metadata field with every document.
- This default versioning is called internal versioning.
- There is also an external versioning type.
- Useful when we want to decide the version of document ourselves.
- Example usage:
```
https://localhost:9200/products/_doc/123?version=11&version_type=external
```
- What is the point of versioning?
  - Tell how many times a document has been modified.
    - This is not useful.
  - Versioning is hardly used anymore, and is mostly a thing from the past.
  - It was previously the way to do optimistic concurrency control.
    - Now there is a better way, though.

### Optimistic Concurrency Control
- ES now uses primary_term and sequence_number to achieve optimistic locking.
- Optimistic locking:
  - Prevent overwriting documents due to concurrent operations.
- As we know, in optimistic locking:
  - First we retrieve the document, and use its primary_term and sequence_number fields.
  - Then when we sent the update request, we send the two values with the update.
    - And if these values are same then only update is done, this is infact what optimistic locking is otherwise ES rejects the write operation.
- Optimistic locking is required for update document incase of concurrent update requests.

How do we handle failures due to error because of optimistic locking?
- Handle the situation at the application level.
  - Retrieve the document.
  - Use _primary_term and _seq_no to create a new update request to retry update.
- If we have processes or threads that might update the same document concurrently then we should consider this approach.

### Updating by query
- This is used to do bulk update i.e. update can affect multiple documents.
- This update query before executing creates a snapshot to do optimistic concurrency control.
- Search queries and bulk requests are sent to replication groups sequentially. ES retries queries upto 10 times. If the query still fails, the whole query is aborted, any changes already made to documents are not rolled back.
- If a document has been modified since taking a snapshot, the query is aborted.
  - This is checked with the document's primary term and sequence number.
- To count version conflicts instead of aborting the query, the conflicts option can be set to proceed.

### Analysis
Analysis
- Text values are analyzed when indexing documents.
- The result is stored in data structures that are efficient for searching, etc.
- When a text value is indexed, an analyser is used to process the text.
- An analyzer consists of three building blocks: character filters, a tokenizer and token filters.
- The result of analyzing text values is then stored in a searchable data structure.

Character filters:
- Adds, removes, or changes characters.
- Analyzers can contain 0 or more character filters.
- Character filters are applied in the order in which they are specified.
- Example (html_strip filter): This filter removes html elements.

Tokenizer:
- An analyzer contains one tokenizer.
- Tokenizes a string i.e. splits into tokens.
- Characters like punctuation, exclamation mark, etc.
- An example could be to split input sentence into words whenever a whitespace is encountered, therefore the input string is tokenised into a number of tokens.

Token filters:
- Receive the output of the tokenizer as input (i.e. the tokens)
- A token filter can add, remove, or modify tokens.
- An analyzer can contain 0 or more token filters.
- Token filters are applied in the order in which they are specified.
- Example: lowercase_filter; this filter lowercases each token in the output.

#### Standard Analyzer
Behaviour of "standard" analyzer which is the default one:
- Character filter is none.
- Tokenizer is standard.
  - Standard tokenizer splits on whitespace, hyphen, etc. and removes punctuations and exclamation marks, etc.
- Token filter is lowercase.
  - This converts every word/token to lowercase.

#### Inverted Indices data structure
- ES stores tokens/terms in several data structures for efficient data access.
- The data structures are handled by Apache Lucene, not ES.
- Inverted indices is one such DS.

Inverted indices:
- Mapping b/w terms and which documents contains them for a given field.
- Outside the context of analyzers, we use the terminology "terms".
- There is one inverted index per text field. Each field has a dedicated inverted index.
- BKD Trees is an optimal DS for numerical data, and acts as their inverted index.

### Mapping
- Defines the structure of documents (e.g. fields and their data types) and how they (tokens) are indexed.
- 2 types of mapping:
  - Explicit mapping: We define field mappings ourselves.
  - Dynamic mapping: ES generates field mappings for us.
  - We can define some mapping ourselves and rest can done by ES, so mapping is quite flexible in ES.

#### Overview of data types
object data type:
- Used for any json object.
- Objects may be nested.
- Denoted using properties parameter.
- Objects are not stored as objects in Apache Lucene.
- These objects are flattened.

nested data type:
- Similar to the object data type, but maintains object relationships, and is useful when indexing array of objects.
- nested objects are stored as hidden documents.

keyword:
- Used for exact matching of values.
- For this data type, the text is left untouched and no tokens are formed.
- Typically used for filtering, aggregations, and sorting.
- For full-text searches, use the text data type instead.
- Fields that use "keyword" data type are analysed using "keyword" analyzer.

#### Type coercion
Data types are inspected when indexing documents.
- They are validated, and some invalid values are rejected.
- E.g. trying to index an object for a text field.
- Sometimes providing a wrong data type is okay.
  - This is where type coercion comes into the picture.
  - ES inspects mapping and coerces into field data type if possible.
  - Example: Sending integer value wrapped in string like "1" works.

Coercion is NOT used for dynamic mapping.
- Supplying "7.4" for a new field will create a text mapping even if string contains a number.
- Always use correct data types.
  - This is essential first time you index a field.
- Coercion is only used as a convenience that forgives us if we use the wrong data type.
- Disabling coercion is a matter of preference.
  - Personally prefer to disable it to ensure that ES receives the data type that it is supposed to; otherwise I prefer to throw an error so that it can be fixed.
  - It is enabled by default. But we can disable it.

#### Arrays
- There is no such thing as array data type.
- Any field may contain 0 or more values.
  - No configuration or mapping needed.
  - In case of text fields in an array, the strings are simply concatenated before being analyzed, and the resulting token are stored within an inverted index as normal.
```
curl --location 'https://localhost:9200/_analyze' \
--header 'Authorization: Basic ZWxhc3RpYzp0SmNIajFoYTlxSnNOKzUwWTRzKw==' \
--header 'Content-Type: application/json' \
--data '{
    "text": [
        "Smartphone",
        "Electronics"
    ]
}'
```
- Array values should be of same data type.
- Remember to use nested data type for array of objects if you need to query the objects independently.

#### Retrieve Mappings
- To retrieve mapping for an index:
```
curl --location 'https://localhost:9200/<INDEX-NAME>/_mapping' \
--header 'Authorization: Basic ZWxhc3RpYzp0SmNIajFoYTlxSnNOKzUwWTRzKw=='
```
- When using dynamic mapping (i.e. not defining the schema explicitly), it might be useful to check how ES has mapped fields based on the documents that have been indexed.
  - Retrieve mapping or schema for a field: https://localhost:9200/reviews/_mapping/field/content
  - Retrieve mapping or schema for a field inside nested object: https://localhost:9200/reviews/_mapping/field/author.email

#### Date
- Three supported formats:
  - A date without time 
  - A date with time 
  - Milliseconds since the epoch (long)
- UTC timezone assumed if none is specified.
- Dates must be formatted according to the ISO 8601 specification.

How date fields are stored?
- Stored internally as milliseconds since the epoch (long)
- Dates are converted to the UTC timezone.
- The same date conversion happens for search queries.

#### How ES treats missing fields?
- All fields in ES are optional.
- You can leave out a field when indexing documents.
- Some integrity checks can be done at app level to check for required fields.
- Adding a field mapping does not make a field required.
- Searches automatically handle missing fields, documents with missing fields as per search query are not considered.

#### Additional parameters for mapping parameters
- The parameters specified in ES can be modified:
  - format: This parameter can be specified for date data type such as epoch_second or Java's DataFormatter.
  - coerce: This parameter can be specified to enable/disable coercion of values (enabled by default).
    - This can be set at field level or at index level.
  - doc_values:
    - Just to give context, inverted indices is excellent for searching text but don't perform well for many other data access patterns.
    - "Doc values" is another data structure used by Apache Lucene.
      - This is optimised for a different data access pattern such as sorting, aggregations and scripting.
      - It is an additional data structure, not a replacement of "inverted indices".
      - ES automatically queries the apt. data structure.
      - This parameter for a field can be disabled to save on disk space.
  - norms:
    - This parameter is used for relevance scoring, often we don't want to filter results, but also rank them.
    - This parameter for a field can be disabled to save on disk space.
      - Useful for fields that won't be used for relevance scoring.

### Reindex
- We cannot update exist mapping for a field. The solution is to reindex documents into a new index.
