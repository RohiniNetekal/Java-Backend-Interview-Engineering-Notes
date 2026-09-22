# Java Backend Engineering Notes

A structured reference for Java Backend Developer / Senior Java Backend interview preparation.

The purpose is to explain concepts deeply enough to discuss **definition, internal working, practical use, trade-offs and production troubleshooting**.

## Core Java

### OOP
Cover encapsulation, inheritance, polymorphism, abstraction, interfaces and composition vs inheritance.

### Collections
Know List/Set/Map, ArrayList, LinkedList, HashMap, HashSet, TreeMap, TreeSet, ConcurrentHashMap and complexity.

### HashMap internals

~~~text
key
 |
hashCode()
 |
hash calculation
 |
bucket
 |
collision handling
 |
equals()
 |
value
~~~

 load factor, resizing, collision behavior, mutable keys and the equals/hashCode contract.

### Streams

Example:

~~~java
Map<String, Double> totals =
    orders.stream()
          .collect(Collectors.groupingBy(
              Order::status,
              Collectors.summingDouble(Order::amount)));
~~~

Know filter, map, flatMap, sorted, distinct, groupingBy, reduce, lazy evaluation and terminal operations.

## Concurrency

Cover:
- Thread/process
- Race conditions
- synchronized
- volatile
- Atomic classes
- ExecutorService
- Callable/Future
- CompletableFuture
- ConcurrentHashMap
- Thread pools
- Deadlock
- Virtual threads

The key backend question is not only "how do I create a thread?" but "what happens when thousands of tasks compete for limited resources?"

## JVM

~~~text
Java source
   |
javac
   |
bytecode
   |
Class Loader
   |
JVM
   +-- Heap
   +-- Thread Stacks
   +-- Metaspace
   +-- JIT
~~~

Important areas:
- Heap vs stack
- Metaspace
- Class loading
- JIT
- Garbage collection
- OutOfMemoryError
- StackOverflowError
- Heap dumps
- Thread dumps

## Spring Boot

Cover:
- IoC and dependency injection
- Constructor injection
- Bean lifecycle
- Component scanning
- Bean scopes
- Configuration
- Profiles
- Auto-configuration
- REST controllers
- Validation
- Global exception handling
- Transactions
- Security

Example constructor injection:

~~~java
@Service
public class MerchantService {
    private final MerchantDao dao;

    public MerchantService(MerchantDao dao) {
        this.dao = dao;
    }
}
~~~

## Hibernate / JPA

Entity lifecycle:

~~~text
Transient -> Managed -> Detached
                 |
                 v
              Removed
~~~

check on:
- Persistence context
- Dirty checking
- First-level cache
- flush
- Lazy/eager loading
- N+1 problem
- Fetch join
- Entity graphs
- Cascade
- Optimistic locking
- Transaction boundaries

## SQL / Oracle

- Joins
- Aggregation
- Subqueries
- CTEs
- Window functions
- Duplicate detection
- Latest row per group
- Running totals
- Top-N
- NULL handling
- Indexes
- Execution plans
- Query tuning

Performance investigation:

Slow API
  -> application timing
  -> SQL timing
  -> execution plan
  -> scan/join/sort analysis
  -> index/statistics/data-volume review
~~~

Do not add an index automatically; understand the query pattern and execution plan first.

## Microservices

Know:
- Service boundaries
- Database ownership
- REST vs messaging
- API gateway
- Timeouts
- Retry/backoff
- Circuit breaker
- Eventual consistency
- Saga
- Outbox pattern
- Distributed tracing

## Kafka

~~~text
Producer -> Topic -> Partition -> Consumer Group
                         |
                       Offset
~~~

 partitions, keys, ordering, offsets, consumer groups, rebalancing, delivery semantics, duplicate processing and idempotent consumers.

## Security

Spring Security:
- Authentication
- Authorization
- Password hashing
- Roles/authorities
- Method security
- CORS/CSRF concepts

JWT:
- Header
- Payload
- Signature
- Expiration
- Issuer/audience
- Access vs refresh tokens

A signed JWT payload is not a secret store; signing does not make the payload encrypted.

## Testing

Understand:

~~~text
Unit test -> isolated class
Integration test -> multiple real components
End-to-end -> complete business flow
~~~

Modern testing topics:
- Testcontainers
- Real database integration
- Kafka integration tests

## Docker / CI/CD

Docker:
- Images
- Containers
- Dockerfile
- Layers
- Ports
- Environment variables
- Volumes
- Networks
- Multi-stage builds
- Health checks


Commit -> Build -> Unit Tests -> Integration Tests -> Package/Image -> Deploy
~~~

## Modern backend topics

Keep building knowledge in:
- Virtual threads
- Structured concurrency concepts
- Kubernetes fundamentals
- AWS
- API gateways
- Resilience patterns
- Profiling
- Load testing

