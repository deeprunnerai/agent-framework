# Passive Observer Agent Template

Template for building **Level 2 Observer Agents** that monitor and track data without blocking application flow.

## Use This Template When

- ✅ Agent observes and reports only
- ✅ No decision-making required
- ✅ Asynchronous, non-blocking operation
- ✅ Need auto-instrumentation
- ✅ Cross-project data aggregation

## Examples

- **Costing Agent**: Track LLM API costs
- **Metrics Agent**: Collect application metrics
- **Usage Agent**: Track feature usage
- **Performance Agent**: Collect performance data

## Architecture

```
Your Application
  ↓ (auto-instrumented)
SDK Agent
  ↓ (buffered, batched)
Central Service
  ↓
Database (aggregation)
```

## Key Characteristics

- **Non-blocking**: Never blocks application flow
- **Fire-and-forget**: Events sent asynchronously
- **Buffered**: Events batched for efficiency
- **Resilient**: Retries, circuit breaker, graceful degradation
- **Auto-registration**: Self-registers with service

## Components Included

### SDK Components
- Configuration Manager
- Lifecycle Manager (init, shutdown, heartbeat)
- Event Buffer (dual triggers: size + time)
- Resilient Transmitter (retry, circuit breaker)
- Auto-instrumentation (optional)
- Graceful error handling

### Service Components
- Registration endpoint
- Ingestion pipeline (batch processing)
- Multi-tenancy (Org → Project → Deployment)
- Time-series storage (PostgreSQL + TimescaleDB)
- Aggregation engine
- Health checks

## Integration Example

### Auto-Instrumentation (Zero Code)
```typescript
// Just require at top of entry file
require('@myorg/my-agent/auto');

// Rest of app - automatically instrumented!
```

### Manual Instrumentation
```typescript
import { MyAgent } from '@myorg/my-agent';

const agent = new MyAgent({
  apiKey: process.env.AGENT_API_KEY,
  projectId: 'my-project'
});

// Track events
agent.track({
  type: 'my_event',
  data: { ... }
});
```

## Performance Targets

- **Tracking overhead**: < 1ms per event
- **Flush latency**: < 500ms
- **Memory usage**: < 50MB for SDK
- **Throughput**: 10,000 events/sec

## Next Steps

1. Copy this template to your new project
2. Customize event schema for your domain
3. Implement auto-instrumentation (if needed)
4. Deploy service and database
5. Integrate SDK into target projects

## Reference Implementation

See `costing-agent` project for full production example.
