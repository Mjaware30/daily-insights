# Daily GitPulse Insights

Last Updated: 2026-04-23T04:49:03.390Z

### Optimization: Mitigating Thundering Herds with Request Collapsing

In high-concurrency environments, a cache miss on a hot key can trigger a **Thundering Herd**—where thousands of concurrent threads attempt to recompute the same expensive value simultaneously, potentially saturating the upstream database.

A senior architectural pattern to solve this is **Request Collapsing** (often implemented via `singleflight`). This ensures that for a unique key, only one execution is active at a time; subsequent callers "subscribe" to the result of the first call rather than spawning redundant I/O.

```go
// Utilizing a SingleFlight group to deduplicate concurrent work
func (s *Service) GetUser(id string) (*User, error) {
    res, err, shared := s.sfGroup.Do(id, func() (interface{}, error) {
        // Only one goroutine enters here; others block on the result
        return s.db.FetchUser(id)
    })

    if err != nil {
        return nil, err
    }
    
    return res.(*User), nil
}
```

By decoupling the request volume from the upstream load, you shift the bottleneck from I/O throughput to simple mutex synchronization—drastically improving tail latency (`p99`) during traffic spikes.