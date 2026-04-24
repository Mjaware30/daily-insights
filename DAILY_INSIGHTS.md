# Daily GitPulse Insights

Last Updated: 2026-04-24T06:06:03.501Z

### Optimization: Request Collapsing for High-Concurrency Services

When scaling read-heavy microservices, a common bottleneck is the **Cache Stampede**. If a popular key expires, multiple concurrent requests may trigger redundant upstream fetches, potentially saturating your database.

To mitigate this, implement **Promise Coalescing** (or Request Collapsing). This ensures that for any unique resource identifier, only one flight is active at a time. Subsequent requests await the resolution of the initial flight rather than initiating their own.

```typescript
const inflight = new Map<string, Promise<Resource>>();

async function getResource(id: string): Promise<Resource> {
  const cached = await cache.get(id);
  if (cached) return cached;

  // Collapse concurrent requests into a single promise
  if (!inflight.has(id)) {
    const work = fetchFromDB(id).finally(() => inflight.delete(id));
    inflight.set(id, work);
  }

  return inflight.get(id)!;
}
```

**Senior Insight:** While this protects your data layer, ensure you implement a timeout on the `inflight` promise to prevent "zombie" requests from blocking the queue if the upstream service hangs. Efficiency is nothing without resilience.