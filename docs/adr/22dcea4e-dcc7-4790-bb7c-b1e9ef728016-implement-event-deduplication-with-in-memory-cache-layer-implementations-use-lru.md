# Implement Event Deduplication with In-Memory Cache Layer: Implementations Use Lru

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The system processes high-frequency events and data streams that may contain duplicate entries due to network retries, multiple event sources, or timing issues
- Without deduplication, downstream consumers would process the same event multiple times, leading to incorrect state, redundant operations, and wasted resources
- An in-memory cache layer provides fast O(1) lookup performance for checking if an event has been seen before, avoiding expensive database queries or external service calls
- The pattern is observed across multiple components (event optimizer, MJPEG hub, keepalive utilities) indicating a cross-cutting architectural concern
- Memory-based caching is appropriate for short-lived deduplication windows where persistence is not required and performance is critical

## Problem Statement

How do we efficiently prevent duplicate event processing in high-throughput systems without introducing significant latency or requiring persistent storage for every deduplication check?

## Decision

1. MAY: Implementations MAY use LRU (Least Recently Used) or other eviction strategies based on access patterns

## Policy Block

- MAY Implementations MAY use LRU (Least Recently Used) or other eviction strategies based on access patterns

In scope:
- Event processing pipelines handling external or user-generated events
- Real-time data stream processors (WebSocket handlers, MJPEG streams, polling mechanisms)
- Keepalive and heartbeat mechanisms that may receive duplicate signals
- Components where duplicate processing would cause incorrect state or side effects

Out of scope:
- Database-level deduplication using unique constraints
- Idempotent operations where duplicate processing is safe and has no side effects
- Single-threaded sequential processors with guaranteed at-most-once delivery
- Batch processing systems where duplicates are handled in post-processing

Exceptions:
- EXC-001: The event processing operation is naturally idempotent and duplicate processing causes no harm
- EXC-002: Memory constraints are severe and alternative deduplication mechanisms (database, external cache) are used

## Rationale

- Pattern detected across 3 files with 90.87% confidence, indicating a consistent architectural approach to event deduplication
- In-memory caching provides sub-millisecond lookup times compared to database queries (10-100ms) or external cache services (1-10ms), critical for high-throughput event processing
- The cache layer acts as a first-line defense against duplicates, reducing load on downstream systems and preventing cascading effects
- Time-based eviction naturally aligns with the temporal nature of duplicate events, which typically arrive within seconds or minutes of each other

## Consequences

Positive:
- Significantly reduced duplicate event processing, improving system correctness and resource efficiency
- Sub-millisecond deduplication checks enable high-throughput event processing without performance degradation
- Lower load on downstream systems (databases, APIs, message queues) due to filtered duplicates
- Simplified debugging and monitoring as duplicate events are caught early in the pipeline

Negative:
- Increased memory footprint proportional to the deduplication window size and event rate
- Cache state is lost on process restart, potentially allowing duplicates during recovery windows
- Additional code complexity for cache management, eviction policies, and monitoring
- Risk of memory leaks if eviction policies are not properly implemented or configured

## Alternatives

- Use database unique constraints for deduplication (rejected)
  Rejected because: Database round-trips add 10-100ms latency per event, unacceptable for high-frequency event streams. Also increases database load and costs.
  When valid: Suitable for low-frequency events or when persistence of deduplication state is required across restarts
- Use external distributed cache (Redis, Memcached) for deduplication (rejected)
  Rejected because: Network latency (1-10ms) and operational complexity of managing external cache infrastructure. Overkill for single-instance deduplication needs.
  When valid: Required for multi-instance deployments where deduplication must work across multiple service instances
- Design all operations to be idempotent and skip deduplication (rejected)
  Rejected because: Not all operations can be made idempotent without significant complexity. Duplicate processing still wastes resources even if safe.
  When valid: Preferred when operations are naturally idempotent and resource waste from duplicates is negligible

## Risks

- Memory exhaustion if cache grows unbounded due to missing or ineffective eviction policies
  Mitigation: Implement mandatory size limits and TTL-based eviction. Monitor cache size metrics and set alerts for abnormal growth.
  Owner: Engineering team
- Duplicate events may slip through during process restarts when cache is empty
  Mitigation: Accept this as a trade-off for performance. Ensure downstream systems can handle occasional duplicates gracefully. Consider warm-up periods after restart.
  Owner: Engineering team
- Cache key collisions if event identifiers are not sufficiently unique
  Mitigation: Use cryptographic hashes or composite keys with sufficient entropy. Validate uniqueness of identifiers in testing.
  Owner: Engineering team

## Implementation Notes

- Use JavaScript Map or Set for cache storage with event IDs or hashes as keys. Consider WeakMap for automatic garbage collection if applicable.
- Implement TTL-based eviction using setTimeout or periodic cleanup intervals. For high-frequency events, consider batch cleanup every N seconds.
- Set cache size limits based on expected event rate and deduplication window. Example: 1000 events/sec × 60 sec window = 60K entries × ~100 bytes = ~6MB memory.
- Add instrumentation to track cache hit rate, size, and eviction frequency. Expose metrics for monitoring and alerting.
- Consider using a library like 'lru-cache' for production-ready LRU implementation with built-in size and TTL management.

## Continuation Context


Verify commands:
- grep -r "new Map\|new Set" --include="*.ts" --include="*.js" | grep -i "cache\|dedup\|seen"
- grep -r "setTimeout\|setInterval" --include="*.ts" --include="*.js" | grep -i "clear\|evict\|cleanup"
- npm test -- --grep "dedup\|duplicate" 2>&1 | grep -E "passing|failing"

Accept when:
- Event processing components use Map or Set data structures to track processed event identifiers
- Cache implementations include eviction logic (TTL-based or size-based) to prevent unbounded growth
- Unit tests verify that duplicate events are correctly identified and filtered by the cache layer

## Enforcement

- Verified by: Code review checklist includes verification of cache eviction policies
- Verified by: Unit tests must demonstrate deduplication behavior with duplicate event scenarios
- Verified by: Performance tests validate cache lookup times remain under 1ms at expected load
- Violation handling: Pull requests without proper cache eviction policies must be rejected
- Violation handling: Production monitoring alerts trigger when cache memory usage exceeds configured thresholds
- Violation handling: Incidents caused by duplicate processing require post-mortem and cache implementation review
- Exception process: Request exception through architecture review board with documented justification
- Exception process: Provide evidence that alternative deduplication strategy is in place or operation is idempotent
- Exception process: Document exception in component README and architecture decision log