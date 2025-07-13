## When Pagination Breaks: Fixing `@EntityGraph` without Breaking Everything

### A Case Study in System-Oriented Tuning for High-Volume Endpoints

> ⚠️ **Disclaimer:** The goal of this write-up is not to showcase Hibernate tuning tricks or JPA annotations. These are well-documented elsewhere. Instead, this article serves as an *entry point* into a broader discussion on architectural design thinking.
>
> By examining one specific bottleneck and its resolution, we aim to highlight how seemingly low-level technical choices often stem from or reflect deeper systemic design tensions. The real takeaway here is not the workaround—but how structured methods and system-level reasoning can lead to more resilient, scalable solutions.
>
> This marks the beginning of a series exploring **systematic design thinking for high-performance backend systems**.

### Problem Summary

High-volume querying on the `/orders` endpoint exposed a series of scalability issues in the production environment. These included inefficient SQL execution patterns, excessive lazy-loading behavior, and ultimately, bottlenecks caused by excessive database round-trips.

One critical problem emerged mid-way through our efforts: attempts to improve load performance via `@EntityGraph` accidentally broke pagination, triggering the obscure yet impactful `HHH000104` warning:

> `HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!`

While this is often treated as a Hibernate quirk, we approached it differently. From a **relational algebra perspective**, pagination is a projection applied after the result set is defined—applying it before entity expansion is both natural and more efficient. This aligns with insights I developed during earlier platform work, particularly from a system called **MatrixOne** (not to be confused with the current open-source database product of the same name, now maintained by MatrixOrigin). That MatrixOne emphasized API designs centered around **OID-first querying**. These lightweight, opaque identifiers made it possible to decouple identity resolution from data hydration, enabling highly efficient random-access reads and clean architectural separation of query phases.

This realization led to a two-phase query model that aligned with how relational systems are meant to behave: first resolve identity (IDs), then hydrate data.

This architecture-driven fix didn’t just restore pagination integrity—it became a gateway to rethinking how APIs, database models, and runtime fetch strategies work together for sustainable performance.

High-volume querying on the `/orders` endpoint exposed a series of scalability issues in the production environment. These included inefficient SQL execution patterns, excessive lazy-loading behavior, and ultimately, bottlenecks caused by excessive database round-trips.

One critical problem emerged mid-way through our efforts: attempts to improve load performance via `@EntityGraph` accidentally broke pagination, triggering the obscure yet impactful `HHH000104` warning:

> `HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!`

While this is often treated as a Hibernate quirk, we approached it differently. From a **relational algebra perspective**, pagination is a projection applied after the result set is defined—applying it before entity expansion is both natural and more efficient.

This realization led to a two-phase query model that aligned with how relational systems are meant to behave: first resolve identity (IDs), then hydrate data.

This architecture-driven fix didn’t just restore pagination integrity—it became a gateway to rethinking how APIs, database models, and runtime fetch strategies work together for sustainable performance.

High-volume querying on the `/orders` endpoint exposed a series of scalability issues in the production environment. These included inefficient SQL execution patterns, excessive lazy-loading behavior, and ultimately, bottlenecks caused by excessive database round-trips.

One critical problem emerged mid-way through our efforts: attempts to improve load performance via `@EntityGraph` accidentally broke pagination, triggering the obscure yet impactful `HHH000104` warning:

> `HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!`

This discovery became the turning point. It prompted a rethinking of how we structure pagination and data loading at a system level, eventually leading to a **two-step fetch strategy** that cleanly separates pagination logic from entity hydration. This key insight—and its architectural implications—became the highlight of our tuning journey.

High-volume querying on the `/orders` endpoint exposed a series of scalability issues in the production environment. These included inefficient SQL execution patterns, excessive lazy-loading behavior, and ultimately, bottlenecks caused by excessive database round-trips.

To systematically address these, we adopted a staged tuning strategy:

1. **Diagnosis** – Use Kibana, database logs, and profiling tools to isolate slow endpoints and identify the cause (e.g. N+1 queries).
2. **Mitigation via eager loading** – Replace runtime lazy-loading with structured eager loading (`EntityGraph`) where applicable. This approach was inspired by guidance from [Baeldung's NamedEntityGraph article](https://www.baeldung.com/spring-data-jpa-named-entity-graphs).
3. **Avoid over-eager pitfalls** – Experiment with subgraphs to eagerly fetch nested structures, but soon ran into Hibernate warnings (see `HHH000104`), echoing common caveats also noted in [JPA EntityGraph practices](https://www.baeldung.com/jpa-entity-graph).
4. **Final tuning with batch fetching** – Apply `@BatchSize` annotations to balance eager loading with batch-controlled fetches to reduce round-trips without causing memory bloat.

Each step built upon the previous one, aiming to balance correctness, performance, and maintainability.

High-volume querying on the `/orders` endpoint exposed a series of scalability issues in the production environment. These included inefficient SQL execution patterns, excessive lazy-loading behavior, and ultimately, bottlenecks caused by excessive database round-trips.

To systematically address these, we adopted a staged tuning strategy:

1. **Diagnosis** – Use Kibana, database logs, and profiling tools to isolate slow endpoints and identify the cause (e.g. N+1 queries).
2. **Mitigation via eager loading** – Replace runtime lazy-loading with structured eager loading (`EntityGraph`) where applicable.
3. **Avoid over-eager pitfalls** – Experiment with subgraphs, and observe when this leads to Hibernate warnings or memory pressure.
4. **Final tuning with batch fetching** – Apply `@BatchSize` to control fetch granularity and reduce round-trips without blowing up memory.

Each step built upon the previous one, aiming to balance correctness, performance, and maintainability.

The `/orders` endpoint suffered from performance issues due to **N+1 query problems** and heavy load on the database server.

- **Trigger:** Kibana logs showed execution time > 5000ms.
- **Example request:**
  ```http
  GET /orders?page=0&size=100000&sortDirection=ASC&sortField=id&includeArchived=false
  ```

### Prior to v2.45.0 (Lazy Fetch)

Before any structured tuning efforts were in place, the default behavior relied heavily on Hibernate's lazy-loading strategy. While theoretically elegant, in practice it caused severe performance degradation due to the classic **N+1 query problem**.

As a workaround, developers inserted deep, imperative logic within `CacheOperations.java` to force eager initialization manually. This logic—duplicated below—was a symptom of the lack of any structured or declarative mechanism to express data shaping.

This kind of solution reflects a common and uncomfortable truth: **when performance problems arise, many developers instinctively reach for 'just-in-time caching' or brute-force loading logic.**

But what’s worse: the developers themselves often know how ugly it is. The code is usually buried in infrastructure services, littered with comments of discontent, yet without better alternatives.

---This block of code, though surrounded by comments that hinted at the developer's own frustration, reflects a common reflex in performance firefighting: **just eagerly load everything, manually**. But while it may 'work', this approach is fundamentally flawed:

- It violates separation of concerns.
- It hardcodes business logic into infrastructure code.
- It is brittle, tightly coupled, and fragile in face of model changes.

> 🧠 More importantly: it reveals that **the default ORM abstractions failed to offer a meaningful alternative**.

This is not about bad code—it's about a lack of architectural primitives for access shaping and control.

```kotlin
// OrderEntity.kt
@get:JsonIgnore
@OneToMany(fetch = FetchType.LAZY, mappedBy = "orderEntity")
var orderItemEntities: MutableSet<OrderItemEntity> = mutableSetOf()

@ManyToMany(fetch = FetchType.LAZY)
var tags: MutableSet<Tag> = mutableSetOf()

@ManyToMany(fetch = FetchType.LAZY)
private var customerEntities: MutableSet<CustomerEntity> = mutableSetOf()
```

```java
// CacheOperations.java (before v2.45.0)
public Page<OrderEntity> getOrders(...) {
    Page<OrderEntity> orderEntityPage = orderRepository.findAll(...);
    for (OrderEntity entity : orderEntityPage.getContent()) {
        for (CustomerEntity customer : entity.getCustomerEntities()) {
            for (AccountEntity acc : customer.getAccountEntities()) {
                for (RegionEntity region : acc.getRegionEntities()) {
                    Hibernate.initialize(region);
                }
            }
        }
        for (Tag tag : entity.getTags()) {
            Hibernate.initialize(tag);
        }
    }
    entityManager.unwrap(Session.class).clear();
    return orderEntityPage;
}
```

> 🧠 This code is not merely unclean—it represents a dead-end in the abstraction. ORM frameworks like Hibernate lack the access shaping capabilities required to express such needs in a structured, evolvable way. Without a dynamic or pluggable access path mechanism, the fallback is always imperative, fragile, and error-prone.

### Optimization with v2.45.0: `@NamedEntityGraph`

One related detail often overlooked: the use of `@Transactional(readOnly = true)` in the controller/resource layer was not primarily for performance, but to **ensure a consistent Hibernate session context**. Lazy-loading requires that the entity remains attached to an open persistence context. Without a transactional boundary, accessing lazy properties outside the controller method would result in:

> `org.hibernate.LazyInitializationException: could not initialize proxy – no Session`

Thus, wrapping the entire `GET /orders` lifecycle in a read-only transaction ensured that JPA/Hibernate could still access and hydrate lazily-fetched collections or proxies when needed, especially before the final shift to eager fetching strategies.

Introduced `@EntityGraph` to resolve lazy-loading with fetch joins.

```kotlin
// OrderEntity.kt
@NamedEntityGraph(
  name = "order-graph",
  attributeNodes = [
    NamedAttributeNode("orderItemEntities"),
    NamedAttributeNode("tags"),
    NamedAttributeNode("customerEntities")
  ]
)
class OrderEntity : OrderEntityFilters
```

```java
// OrderRepository.java
@EntityGraph(value = "order-graph")
Page<OrderEntity> findAll(Predicate predicate, Pageable pageable);
```

### Attempt with SubGraph (not recommended)

This approach was guided by Baeldung's deeper EntityGraph and Subgraph usage patterns, aiming to resolve deeper nesting via structured graph configurations. However, it triggered performance issues due to Hibernate's handling of pagination and collection fetch joins.

```kotlin
@NamedEntityGraph(
  name = "order-graph",
  attributeNodes = [
    NamedAttributeNode("orderItemEntities"),
    NamedAttributeNode("tags"),
    NamedAttributeNode(value = "customerEntities", subgraph = "customers-accounts")
  ],
  subgraphs = [
    NamedSubgraph(name = "customers-accounts", attributeNodes = [
      NamedAttributeNode("accountEntities")
    ])
  ]
)
```

This solution led to further in-memory issues, most notably the infamous `HHH000104` warning from Hibernate:

```
HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!
```

This warning indicates that pagination couldn't be applied at the database level due to collection fetch joins, causing the result set to be materialized and sliced in memory — leading to high memory consumption and unpredictable performance under load.

🔍 **Root Cause Insight:** According to Baeldung, EntityGraphs with nested collections are powerful but risky when paired with pagination, especially for `@OneToMany` or `@ManyToMany` joins.

📌 **Resolution Strategy:** Shift away from subgraph-based eager loading for paginated endpoints and instead adopt `@BatchSize` to retrieve associated entities in grouped queries.



### The Turning Point: Pagination vs. Fetch Graphs

The real breakthrough in this tuning journey came not from tuning parameters or annotations—but from deeply understanding a hidden contradiction: **pagination and collection fetch joins do not coexist safely in JPA/Hibernate**.

Initially, we leaned on `@EntityGraph` to eagerly fetch related entities—clean and declarative. However, this collided with pagination logic and surfaced the infamous `HHH000104` warning:

```
HHH000104: firstResult / maxResults specified with collection fetch; applying in memory!
```

This wasn’t just a technical nuisance—it was an architectural signal. In-memory pagination isn’t scalable. The system needed a structural rethink.

### The Final Solution: Two-Step Fetching

Inspired by a lesser-known workaround (PS-6176), we implemented a clean separation of pagination from loading:

1. **Step 1:** Fetch paginated root entities by ID only (without any collection joins).
2. **Step 2:** Fetch the full graph of associations using `WHERE id IN (...)`, followed by restoring original pagination order.

This approach removed the reliance on `EntityGraph` in pagination contexts while preserving performance and correctness.



This rethinking of the data-loading model became a replicable pattern—not just a fix for this one endpoint, but a foundational insight into how API shape and DB strategy must align.

The real turning point came from resolving the subtle conflict between **collection fetch joins** and **pagination**.

Although `@EntityGraph` provided a clean way to eagerly fetch associated entities, it silently broke pagination logic—triggering `HHH000104`. This is not just a Hibernate quirk but a critical architectural trade-off.

> The breakthrough was in recognizing that pagination and deep collection fetching simply **cannot coexist reliably** at the database level.





The ultimate fix—highlighted in the internal tuning effort (PS-6176)—was to **split the query into two SQL operations**:

1. **Step 1:** Perform a lightweight paginated query on the root entity (`OrderEntity`) **without** `@EntityGraph`.
2. **Step 2:** Issue a second query to fetch the full list of associated entities **using the IDs** from Step 1.

This preserves server-side pagination, avoids `HHH000104`, and retrieves complete entity graphs **without triggering in-memory slicing**.

> 🔧 *Bonus Tip:* Re-sort the result set after second fetch to preserve the original pagination order.

This solution avoids native queries, is portable, and maintains consistency with the rest of the JPA stack.

The real turning point came from resolving the subtle conflict between **collection fetch joins** and **pagination**.

Although `@EntityGraph` provided a clean way to eagerly fetch associated entities, it silently broke pagination logic—triggering `HHH000104`. This is not just a Hibernate quirk but a critical architectural trade-off.

> The breakthrough was in recognizing that pagination and deep collection fetching simply **cannot coexist reliably** at the database level.

Instead of accepting in-memory pagination (which breaks performance under load), the final solution preserved server-side pagination by **deliberately removing nested collection joins from the EntityGraph** and instead:

- used `@BatchSize` to prefetch relations in a scalable way
- ensured pagination operates purely on the root entity (`OrderEntity`)

This balance between pagination integrity and association loading became the key insight—and perhaps the most replicable pattern from this experience.

### Next Step: Use `@BatchSize`

```kotlin
// AccountEntity.kt
@BatchSize(size = 20)
var regionEntities: MutableSet<RegionEntity> = HashSet()

// CustomerEntity.kt
@BatchSize(size = 20)
private var accountEntities: MutableSet<AccountEntity> = HashSet()
```

### Load Testing & Metrics

- K6 load testing on staging environment.
- Local DB metrics collected.

### Acknowledgement & Intention

Much of the tuning strategy described above—especially the use of `@NamedEntityGraph` and the cautionary note around Hibernate's `HHH000104`—was inspired by Baeldung’s excellent articles. The links have been included in the references section below.

> 📝 *Note:* These ideas are not original. In fact, the `HHH000104` issue is obscure enough that the fix isn’t widely covered. It took effort to find those resources, and that’s why I want to share them here, hoping to help others avoid similar struggles.

This article is not a tutorial, but a stepping stone. It marks the start of a broader discussion on **how we think about performance tuning, data shaping, and API-layer efficiency**. The real motivation is not the fix, but the **design dilemma** it reveals. That design reflection will be the subject of the follow-up articles in this series—a humble proposal of sorts.

### Looking Ahead: From Fixes to Foundations

While the two-step fetch strategy effectively resolved the pagination conflict and eliminated `HHH000104` warnings, one lingering observation remains difficult to ignore:

> Even in the optimized version, metrics revealed a peak of **2,450 concurrent cursors** in production.

This number is more than an operational curiosity—it’s a sign of architectural tension. Such a high count of active cursors is not sustainable long-term and raises questions about system robustness under pressure.

It also reveals a deeper structural limitation: **most ORM frameworks, including Hibernate, lack a pluggable or extensible access path abstraction**. Their data loading strategies—whether eager or lazy—are encoded in static annotations and implicit resolution paths. These built-in defaults often fail to accommodate high-performance scenarios, especially in systems with layered domain boundaries and variable access patterns. More critically, because these access paths are static, **they cannot be influenced at runtime**—preventing the system from adapting query plans based on evolving access context, load conditions, or consumer intent.

Beyond that, the **query process itself lacks structure**—there is no formal abstraction around how queries are constructed, routed, or decomposed. The underlying **data model was never intended for performance-driven access**, and often fails to express intent, optimize resolution order, or support partial shaping strategies. And perhaps most troubling: the process is **not observable**—there is no way to trace which access paths were used, where time was spent, or why a given hydration strategy was chosen.

These are not minor details—they are violations of what we believe to be **foundational qualities of good software architecture**:

- Structure
- Observability
- Adaptability
- Intentional design of data flow

> 🧭 This is where performance tuning meets architecture. And it’s where this series is headed next: toward a **systematic exploration of high-performance design**—not just as a technical goal, but as a product of principled structure.

While the two-step fetch strategy effectively resolved the pagination conflict and eliminated `HHH000104` warnings, one lingering observation remains difficult to ignore:

> Even in the optimized version, metrics revealed a peak of **2,450 concurrent cursors** in production.

This number is more than an operational curiosity—it’s a sign of architectural tension. Such a high count of active cursors is not sustainable long-term and raises questions about system robustness under pressure.

It also reveals a deeper structural limitation: **most ORM frameworks, including Hibernate, lack a pluggable or extensible access path abstraction**. Their data loading strategies—whether eager or lazy—are encoded in static annotations and implicit resolution paths. These built-in defaults often fail to accommodate high-performance scenarios, especially in systems with layered domain boundaries and variable access patterns. More critically, because these access paths are static, **they cannot be influenced at runtime**—preventing the system from adapting query plans based on evolving access context, load conditions, or consumer intent.

So we must ask:

- What’s driving this accumulation of cursors?
- Is this an artifact of Hibernate's fetch semantics—or a design compromise made upstream?
- Can we influence how access paths are selected dynamically?
- Or must we break free from ORM conventions to reclaim control over data shape and lifecycle?

These aren’t questions tuning alone can answer. They touch on how the system is **structured**, how responsibilities are **separated**, and how data is **contracted and delivered**.

> 🧭 This is where performance tuning meets architecture. And it’s where this series is headed next: toward a **systematic exploration of high-performance design**—not just as a technical goal, but as a product of principled structure.

While the two-step fetch strategy effectively resolved the pagination conflict and eliminated `HHH000104` warnings, one lingering observation remains difficult to ignore:

> Even in the optimized version, metrics revealed a peak of **2,450 concurrent cursors** in production.

This number is more than an operational curiosity—it’s a sign of architectural tension. Such a high count of active cursors is not sustainable long-term and raises questions about system robustness under pressure.

So we must ask:

- What’s driving this accumulation of cursors?
- Is this an artifact of Hibernate's fetch semantics—or a design compromise made upstream?
- Are we unintentionally stretching the transaction boundaries, data lifecycles, or resource scoping?

These aren’t questions tuning alone can answer. They touch on how the system is **structured**, how responsibilities are **separated**, and how data is **contracted and delivered**.

> 🧭 This is where performance tuning meets architecture. And it’s where this series is headed next: toward a **systematic exploration of high-performance design**—not just as a technical goal, but as a product of principled structure.

While the immediate performance gains are clear, the story doesn't end here.

> Even with the optimized two-step fetching strategy, metrics revealed something unsettling: **2,450 active cursors** in production at peak.

This isn’t just a number—it’s a latent risk. A ticking time bomb. Under certain usage spikes or degraded conditions, this could lead to database resource exhaustion, degraded user experience, or even outages.

So we must ask:

- What is the true source of this excessive cursor count?
- Is it a matter of poor cursor management in Hibernate?
- Or is it a symptom of a deeper design issue in how we shape and deliver data?

These questions can’t be answered with tuning alone. They require stepping back and evaluating the **system’s data contract**, **responsibility boundaries**, and **resource lifecycle strategy**.

👉 That is where this article points next: toward a structured approach to architecture that aligns data flow, performance, and maintainability.

---

### References

- Internal: [p6spy](https://confluence.se.axis.com/display/PUBT/p6spy)

- Tuning JPA: [JPA Parameters](https://confluence.se.axis.com/display/PUBT/Useful+JPA+Parameters)

- Guides:

  - [Baeldung on EntityGraph](https://www.baeldung.com/spring-data-jpa-named-entity-graphs)
  - [JPA EntityGraph (advanced usage)](https://www.baeldung.com/jpa-entity-graph)
  - [Appsloveworld – Using JPA EntityGraph with lazy load and pagination issues (HHH000104)](https://www.appsloveworld.com/springboot/100/123/using-spring-data-jpa-entitygraph-with-lazy-load-mode-for-namedattributenode-fiel?utm_content=cmp-true)

- Internal: [p6spy](https://confluence.se.axis.com/display/PUBT/p6spy)

- Tuning JPA: [JPA Parameters](https://confluence.se.axis.com/display/PUBT/Useful+JPA+Parameters)

- Guides: [Baeldung on EntityGraph](https://www.baeldung.com/spring-data-jpa-named-entity-graphs), [JPA EntityGraph](https://www.baeldung.com/jpa-entity-graph)

