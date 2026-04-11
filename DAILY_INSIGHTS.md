# Daily GitPulse Insights

Last Updated: 2026-04-11T06:08:00.432Z

### Optimization: Mitigating Thundering Herds with Request Collapsing

In high-throughput systems, a cache miss on a hot key can trigger a "thundering herd" effect—hundreds of concurrent requests hitting the upstream database simultaneously. To prevent resource exhaustion, implement a **SingleFlight** pattern.

By utilizing a coordination primitive, you ensure that only the first request executes the expensive I/O operation. Subsequent concurrent callers subscribe to the result of the initial inflight request rather than initiating their own.

```go
// Simplified SingleFlight pattern for concurrent I/O
func (s *Service) GetData(ctx context.Context, key string) (Data, error) {
    s.mu.Lock()
    if call, ok := s.inflight[key]; ok {
        s.mu.Unlock()
        return call.Wait() // Block on existing inflight request
    }
    
    call := newCall()
    s.inflight[key] = call
    s.mu.Unlock()

    res, err := s.fetchFromDB(key)
    call.Done(res, err) // Broadcast result to all waiters
    
    s.mu.Lock()
    delete(s.inflight, key)
    s.mu.Unlock()

    return res, err
}
```

This pattern shifts the load from the database to the application's memory, significantly reducing tail latency and preventing upstream saturation during TTL expirations.