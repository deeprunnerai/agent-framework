# Hybrid Agent Template

Template for building agents that combine **observation** and **control** capabilities.

## Use This Template When

- ✅ Agent both observes AND makes decisions
- ✅ Mixed blocking and non-blocking operations
- ✅ Need feedback loops (observe → decide → observe)
- ✅ Adaptive behavior based on observations

## Examples

- **Cache Agent**: Observes access patterns → Makes caching decisions
- **Scaling Agent**: Monitors performance → Scales resources
- **Deployment Agent**: Tracks metrics → Decides to rollback
- **Adaptive Router**: Observes latency → Routes traffic
- **Circuit Breaker Agent**: Monitors errors → Opens/closes circuits

## Architecture

```
Your Application
  ↓
SDK Agent
  ├─ Observer Path (async) ──► Central Service ──► Analytics
  │                                                  ↓
  │                                              Intelligence
  │                                                  ↓
  └─ Controller Path (sync) ─► Decision Engine ←────┘
                                    ↓
                                Decision
```

## Key Characteristics

- **Dual Mode**: Both observer and controller capabilities
- **Feedback Loop**: Observations inform decisions
- **Adaptive**: Behavior changes based on patterns
- **Mixed Sync/Async**: Some operations block, others don't

## Components Included

### SDK Components
**Observer Components**:
- Event Buffer (for observations)
- Transmitter (async sending)
- Auto-instrumentation (optional)

**Controller Components**:
- Middleware/Dependencies
- Local Validation Cache
- Decision Client

**Shared Components**:
- Configuration Manager
- Lifecycle Manager
- Health Checks

### Service Components
**Observer Components**:
- Ingestion Pipeline
- Time-series storage
- Pattern detection

**Controller Components**:
- Decision endpoint
- Policy management
- Validation

**Intelligence Components**:
- Pattern Analysis (learns from observations)
- Decision Engine (uses learned patterns)
- Feedback Loop (improves over time)

## Integration Example

### Cache Agent Example

```typescript
import { CacheAgent } from '@myorg/cache-agent';

const cacheAgent = new CacheAgent({
  apiKey: process.env.CACHE_AGENT_API_KEY,
  projectId: 'my-project'
});

// Observer: Track cache access patterns (async, non-blocking)
function getCachedData(key: string) {
  const value = cache.get(key);

  // Async tracking - doesn't block
  cacheAgent.track({
    type: 'cache_access',
    key,
    hit: value !== undefined,
    timestamp: Date.now()
  });

  return value;
}

// Controller: Agent decides what to cache (sync, blocking)
async function shouldCache(key: string, data: any): Promise<boolean> {
  // Agent makes decision based on learned patterns
  const decision = await cacheAgent.shouldCache({
    key,
    size: Buffer.byteLength(JSON.stringify(data)),
    accessPattern: await cacheAgent.getAccessPattern(key)
  });

  return decision.cache; // true/false
}

// Use both together
async function getData(key: string) {
  // Try cache first (observer tracks)
  let data = getCachedData(key);

  if (!data) {
    // Fetch from source
    data = await fetchFromSource(key);

    // Ask agent if we should cache (controller decides)
    if (await shouldCache(key, data)) {
      cache.set(key, data);
    }
  }

  return data;
}
```

### Deployment Agent Example

```typescript
import { DeploymentAgent } from '@myorg/deployment-agent';

const deployAgent = new DeploymentAgent({
  apiKey: process.env.DEPLOY_AGENT_API_KEY,
  projectId: 'my-service'
});

// Observer: Track deployment metrics (async)
setInterval(() => {
  deployAgent.track({
    type: 'metrics',
    errorRate: getErrorRate(),
    latency: getLatencyP95(),
    throughput: getThroughput()
  });
}, 10000); // Every 10 seconds

// Controller: Decide on rollback (sync, critical)
async function checkDeploymentHealth(): Promise<void> {
  const decision = await deployAgent.evaluateDeployment();

  if (decision.action === 'rollback') {
    console.error('Agent recommends rollback:', decision.reasoning);

    if (decision.autoRollback) {
      // Agent autonomously rolls back
      await performRollback();
    } else {
      // Alert humans
      await alertTeam(decision);
    }
  }
}
```

## Design Patterns

### Pattern 1: Observe → Learn → Decide

```
1. Observer tracks events (async)
   ↓
2. Intelligence layer learns patterns
   ↓
3. Controller uses patterns to make decisions (sync)
```

Example: Cache agent learns access patterns, decides what to cache

### Pattern 2: Decide → Observe → Adapt

```
1. Controller makes decision (sync)
   ↓
2. Observer tracks outcome (async)
   ↓
3. Intelligence adapts policy based on outcomes
```

Example: Deployment agent deploys, tracks metrics, adapts rollback policy

### Pattern 3: Continuous Feedback Loop

```
Observe ──► Learn ──► Decide ──► Act
   ▲                              │
   └──────────────────────────────┘
        (Observe outcome)
```

Example: Adaptive rate limiter adjusts limits based on observed behavior

## Performance Targets

**Observer Path** (Non-blocking):
- Tracking overhead: < 1ms
- Buffering: Batch every 10s or 100 events
- Fire-and-forget: Never blocks

**Controller Path** (Blocking):
- Decision latency: p95 < 100ms
- Cache hit rate: > 90%
- Must be fast - in critical path!

## Intelligence Level

Hybrid agents typically use **Level 3: Behavioral Learning**:

- Learns patterns from observations
- Adapts thresholds dynamically
- Uses learned patterns for decisions
- Improves over time

## Use Cases by Domain

### Performance Optimization
- **Cache Agent**: Learn access patterns → Cache intelligently
- **Scaling Agent**: Monitor load → Auto-scale
- **Load Balancer Agent**: Track latency → Route optimally

### Reliability
- **Circuit Breaker Agent**: Count errors → Open/close circuits
- **Deployment Agent**: Monitor metrics → Rollback/proceed
- **Health Check Agent**: Track health → Route traffic

### Resource Management
- **Connection Pool Agent**: Track usage → Adjust pool size
- **Memory Agent**: Monitor allocation → Trigger GC
- **Thread Pool Agent**: Track utilization → Resize pool

## When NOT to Use Hybrid

❌ **Use passive-observer instead if**:
- No decisions needed
- Pure monitoring/tracking
- No blocking operations

❌ **Use active-controller instead if**:
- No observations needed
- Pure policy enforcement
- Static rules (no learning)

✅ **Use hybrid if**:
- Need both observation AND control
- Decisions informed by observations
- Adaptive behavior required
- Feedback loops needed

## Implementation Sequence

**Phase 1**: Start with Observer
1. Build observer components
2. Collect data
3. Build analytics

**Phase 2**: Add Intelligence
1. Pattern detection
2. Behavioral learning
3. Prediction models

**Phase 3**: Add Controller
1. Decision endpoint
2. Policy enforcement
3. Action execution

**Phase 4**: Close the Loop
1. Track decision outcomes
2. Improve predictions
3. Adapt policies

## Next Steps

1. Copy this template to your new project
2. Identify observation points
3. Define decision logic
4. Implement feedback loop
5. Test adaptive behavior
6. Monitor improvements over time

## Reference Examples

- Adaptive cache systems
- Auto-scaling platforms
- Canary deployment systems
- Circuit breaker implementations
