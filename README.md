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



## Redis & Caching

Caching is a performance optimization, but it introduces consistency and failure-handling decisions. A strong backend design should explain the cache policy, not simply say "use Redis".

### Cache-aside

The application checks Redis first. On a miss it reads the database, stores the result with a TTL, and returns it.

```text
Request -> Service -> Redis
                    |   |
                  HIT  MISS
                    |   |
                    |   v
                    | Database
                    |   |
                    +<--+
```

For a write, a common baseline is to update the database successfully and then evict the corresponding cache entry. The next read repopulates it.

### Spring example

```java
@Cacheable(cacheNames = "merchant", key = "#merchantId")
public MerchantResponse getMerchant(Long merchantId) {
    return repository.findById(merchantId)
            .map(this::toResponse)
            .orElseThrow(() -> new MerchantNotFoundException(merchantId));
}

@CacheEvict(cacheNames = "merchant", key = "#merchantId")
public void updateMerchant(Long merchantId, UpdateMerchantRequest request) {
    // validate and update database
}
```

### Key and TTL design

Prefer deterministic, versioned keys such as:

```text
merchant:v1:101
merchant:v1:102
```

TTL should reflect the business tolerance for stale data. Longer TTL improves hit rate but increases the stale-data window; shorter TTL improves freshness but causes more misses.

### Cache stampede

If a popular key expires and thousands of requests miss at once, all of them may hit the database. Learn TTL jitter, request coalescing/single-flight, controlled refresh and carefully scoped locking.

### Cache penetration and hot keys

Repeated requests for nonexistent data can cause repeated database misses. Short-lived negative caching, validation and rate limiting can help. A hot key can also overload a single cache path and may require local caching, request coalescing or another scaling strategy.

### Redis failure

If Redis is an optimization rather than the source of truth, a cache outage may fall back to the database. But a large simultaneous fallback can overload the database. Therefore define Redis timeouts, database capacity, rate limits, circuit breaking/degradation and monitoring before calling the design production-ready.

### Observability

Track at least:
- cache hit/miss rate
- Redis latency and errors
- evictions and memory usage
- hot keys
- database load before/after caching

### Testing

Test cache hits, misses, invalidation and failure behavior. For integration tests, Testcontainers can run a real Redis instance so serialization, TTL and actual Redis interactions are exercised rather than only mocked.

### Common interview questions

1. How does cache-aside work?
2. TTL vs invalidation?
3. How does cache stampede happen?
4. What happens to the database if Redis goes down?
5. How would you design a cache key?
6. When should you avoid caching?
7. How would you test Redis integration?

### Practical exercises

- [ ] Add Redis cache-aside to a Spring Boot read API
- [ ] Configure TTL and cache eviction on updates
- [ ] Add a Testcontainers Redis integration test
- [ ] Measure cache hit/miss behavior
- [ ] Simulate Redis failure and document the fallback strategy
