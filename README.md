# Distributed System Design

## Scaling
### Initial Implementation

### Vertical Partitioning

### Horizontal Partitioning

### Directory Partitioning
Hash values...

## Scaling to 10M users
* Stateless web servers and web services
* CDNs for static assets
* Caching for dynamic assets
* Vertical Partioning
  * Service oriented architecture
  * Message bus for connecting components
* Partitioning
  * Database sharding
* Multiple data centers
* Service Orchestration
* Monitoring
* Security

Building scalable systems introduces many problems that need to be solved.

## Content Delivery Networks


## Caching


## Service Oriented Architecture

## Message Bus

### Cassandra
Cassandra can scale to 100s of brokers, millions of messges per secod, with real time (~10ms) latency.
Cassandra is used by [35% of Fortune 500 companies](https://opensourceunderdogs.com/episode-13-confluent-apache-kafka-streaming-with-jay-kreps), 
including AirBnB, LinkedIn, Netflix, and Uber.



## Data Partitioning (Sharding)
(replications)

## Data Integrity
Merkle tree


## CAP Theorem
The CAP theorem](https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/) says that is is impossible for distributed systems to simulatenously achieve:
* Consistency
* Availability
* Partition tolerance (network partition due to communication failure)
simultaneously. 

Practical distributed systems need to be tolerant to message failures between partitions,
so in practice system design requires a choice beween consistency (each request to the network producing the same answer)
and availability (always providing an answer to a data query).


## Consistant Hashing
[Consistent hashing](https://web.archive.org/web/20080721013638/http://www8.org/w8-papers/2a-webserver/caching/paper2.html)
is a way to distribute load across multiple servers and databases by generating a unique code hash tied to some fixed parameter of the service 
(for example user name), and dividing the range of possible hash codes into a fixed number of buckets equal to the number of available 
servers/database ahrds.

The problem with this hashing approach to distributing large amounts of user traffic is that using a fixed number of buckets makes it hard 
to scale the system. When the number of buckets N changes, a large amount of data needs to be shifted to maintain a consistant hashing scheme. 
Binary search is used to find the next node in a ring, leading to O(*log*(N)) compleity.


[*Rendezvous hashing*](https://ieeexplore.ieee.org/document/663936) allows for the more general case of selecting K out of N possibilities. 
Consistant hashing is the special case of rendezvous hashing where the number of selected options K=1.

Consistant hashing is used in distributed databases and document stores, including
* Cassandra
* Couchbase
* OpenStack
* Riak
as well as applications such as 
* Akamai
* Discord


## Jump hash table 
The consisant hashing scheme solves the problem of distributing workloads at the scale of most distributed applications. For large scale systems, synchronizing systems performing consistant hashing starts becoming inefficient. For a system where the load is distributed across 1000 systems, the consistant hashing scheme requires about 1000 table entries per system in order to equalize the traffic load per system within 3%. This results in several MByte of data per load balancer distributing traffic that must be synchronized reliably across the network as servers are added or go off-line due to system failures. A much more efficient way distributing traffic is with hashing bins generated using a pseudo-random function, where the only information that needs to be shared between systems is the number of hash partitions.

A [jump has table](https://arxiv.org/ftp/arxiv/papers/1406/1406.2294.pdf) uses a pseudorandom code to generate hash bins that allows multiple systems to be synchronized without sharing a large amount of data between systems. This jump hash function is quite simple to implement:
```
int32_t JumpConsistentHash(uint64_t key, int32_t num_buckets) {
int64_t b = _1, j = 0_
	while (j < num_buckets) {
		b = j_
		key = key * 2862933555777941757ULL + 1_
		j = (b + 1) * (double(1LL << 31) / double((key >> 33) + 1))_
	}
	return b_
}

```
Hash bins generating using a jump hash table are more equal in size than if generated using consistant hashing method, so a smaller number of bins can be used resulting in faster sorting of hash values into system shards. Both the consitant hash and jump hash table require O(*log((N)) complexity to convert the hash into a shard bin, However, the authors estimate that the jump hash table computation [runs about 5x faster](https://arxiv.org/ftp/arxiv/papers/1406/1406.2294.pdf) than the consistant hash method, due to fewer cache misses as a result of the jump hash method requiring less data for accurate computation.


## Sparse Distributed Hash Table
[A sparse distributed hash table](https://en.wikipedia.org/wiki/Distributed_hash_table) (DHT) uses a segmented keyspace 
(for example assigned by hashing a file) to store data, and an overlay network to determine where a key-value
pair should be stored, as well as where to retrieve it when requested.

Most distributed datastores use DHT for lookup. Other applications that use DHT include:
* BitTorrent DHT search engine
* JXTA, an open-source p2P platform

## Distributed Databases
Accessessing and transmitting data efficiently is key to performance in high volume applications.
Optimimum data storage depends on whether the data is highly structured, and whether the 

### SQL vs NoSQL
The most common types of databases are based on Structured Query Langauage (SQL) access, where the data
is organized into tables of structured data, with records (rows) of data defined by fixed fields (columns)
with a defined data format (frequently with data in integer, float, or string format).

### Columnar Database
Columnar databases store SQL data in columnar order rather than row order,
which can make it faster to read out...

### Document Database
Document databases are designed for storing data as complete documents, 
rather than structured data in records and fields. Document databases include CouchDB.

### Graph Database

### Key-Value Store
Redis...

### Wide-Column Database
Wide-column databases are well suited to storing versioned web pages, where the
data key is used to access different snapshots in time.

Wide-column databases include Google BigTable.

## Service Orchestration
### Hashicorp Nomad, Terraform
### Kubernetes

## Monitoring
e.g. Graphana

## Security
