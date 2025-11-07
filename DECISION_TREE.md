# Decision Tree: Building Your Agent

A step-by-step framework for making architectural decisions when building an agent.

## How to Use This Guide

1. **Answer questions sequentially** - Each answer guides the next decision
2. **Document your choices** - Record decisions for future reference
3. **Iterate** - Start simple, add complexity as needed
4. **Validate** - Use the validation checklist at the end

---

## Step 1: Problem Definition

### Question 1.1: What problem are you solving?

**Action**: Write a one-sentence problem statement

**Example Answers**:
- "Track LLM API costs across all projects"
- "Provide intelligent authentication with threat detection"
- "Monitor application performance and detect anomalies"
- "Coordinate deployments with auto-rollback"

**Output**: Problem statement documented

---

### Question 1.2: Why not use a traditional service?

Check all that apply:

- [ ] Need autonomous behavior (acts without explicit invocation)
- [ ] Need learning/adaptation over time
- [ ] Need proactive actions based on observations
- [ ] Cross-cutting concern affecting multiple projects
- [ ] Continuous monitoring or enforcement required
- [ ] Agent patterns provide significant developer experience improvements

**Decision**:
- **0-1 checked**: Consider traditional service instead
- **2-3 checked**: Agent is appropriate
- **4+ checked**: Agent is strongly recommended

---

## Step 2: Agent Classification

### Question 2.1: What is the autonomy level needed?

```
Choose ONE:

[ ] Level 1: Instrumented
    - Just passive data collection
    - No decisions
    → Use: Simple logging, basic metrics

[ ] Level 2: Reactive
    - Automated collection + self-management
    - Buffer management, auto-registration
    → Use: Cost tracking, basic monitoring
    → Template: passive-observer

[ ] Level 3: Proactive
    - Pattern detection + alerting
    - Recommendations
    → Use: Performance monitoring, capacity alerts
    → Template: hybrid

[ ] Level 4: Autonomous
    - Independent decisions + learning
    - Auto-remediation
    → Use: Security, fraud detection
    → Template: active-controller + intelligence

[ ] Level 5: Collaborative
    - Multi-agent coordination
    - Emergent behavior
    → Use: Complex orchestration (advanced)
    → Template: Custom
```

**Output**: Autonomy level = [1-5]

---

### Question 2.2: Does the agent block application flow?

```
Choose ONE:

[ ] Observer (Non-blocking)
    - Agent observes but doesn't affect app
    - Fire-and-forget
    - Asynchronous operation
    → Example: costing-agent, metrics-agent

[ ] Controller (Blocking)
    - Agent makes decisions that affect app
    - Request/response
    - Synchronous operation
    → Example: auth-agent, rate-limit-agent

[ ] Hybrid (Both)
    - Observes AND controls based on context
    - Mixed sync/async
    → Example: cache-agent, adaptive-router
```

**Output**: Operation mode = [Observer/Controller/Hybrid]

---

### Question 2.3: What intelligence level is needed?

See [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) for details.

```
Choose ONE:

[ ] Level 0: Pure Instrumentation
    - Collects raw data only
    - No processing

[ ] Level 1: Aggregation
    - Statistical summaries
    - Cost calculations
    → Example: costing-agent

[ ] Level 2: Pattern Detection
    - Anomaly detection
    - Threshold alerts
    → Example: performance-agent

[ ] Level 3: Behavioral Learning
    - Learns patterns
    - Builds profiles
    - Adapts thresholds
    → Example: adaptive agents

[ ] Level 4: Autonomous Decision-Making
    - Risk scoring
    - Independent actions
    - Self-improvement
    → Example: auth-agent
```

**Output**: Intelligence level = [0-4]

---

## Step 3: Architecture Selection

### Question 3.1: What deployment model?

```
Choose ONE:

[ ] SDK-Only
    Pros: No infrastructure, simplest
    Cons: No central visibility
    Use when: Single project, no aggregation

[ ] Service-Only
    Pros: Centralized control
    Cons: Higher latency, no local optimization
    Use when: Pure server-side decisions

[ ] Hybrid (Service + SDK) ⭐ RECOMMENDED
    Pros: Best of both worlds
    Cons: More complexity
    Use when: Need both local and central capabilities
```

**Recommendation**:
- **Observer agents**: Hybrid (SDK collects, service aggregates)
- **Controller agents**: Hybrid (SDK validates locally, service decides)
- **Simple agents**: SDK-only if no aggregation needed

**Output**: Deployment = [SDK-Only/Service-Only/Hybrid]

---

### Question 3.2: What communication pattern?

```
Based on your operation mode:

If OBSERVER:
  [ ] Push Model (SDK → Service)
      - SDK pushes events
      - Buffered, batched
      → Use: Event streaming

If CONTROLLER:
  [ ] Request/Response (Bidirectional)
      - SDK requests, waits for response
      - Synchronous
      → Use: Validation, authorization

If HYBRID:
  [ ] Mixed (Push for events, Pull for validation)
      - Both patterns
      → Use: Complex agents
```

**Output**: Communication = [Push/Pull/Request-Response/Mixed]

---

### Question 3.3: What data flow model?

```
Choose based on blocking behavior:

[ ] Synchronous
    - Application waits for response
    - Required for: Controllers
    - Latency budget: < 50ms

[ ] Asynchronous
    - Application continues immediately
    - Required for: Observers
    - Latency flexible: < 1s

[ ] Buffered
    - Agent batches before sending
    - Efficient for high volume
    → Use: Event tracking
```

**Output**: Data flow = [Sync/Async/Buffered]

---

## Step 4: Component Selection

### Question 4.1: What SDK components are needed?

**Core Components** (Always required):
- [x] Configuration Manager
- [x] Lifecycle Manager (init, shutdown)
- [x] Communication Layer (HTTP client)

**For Observer Agents**:
- [ ] Event Buffer (batching)
- [ ] Transmitter (resilient sending)
- [ ] Instrumentation Layer (auto-capture)

**For Controller Agents**:
- [ ] Validation Layer (request validation)
- [ ] Cache Layer (local token validation)
- [ ] Middleware/Dependencies (Express/FastAPI)

**For Intelligent Agents** (Level 3-4):
- [ ] Local decision engine (optional)
- [ ] Behavioral cache

**Output**: List of SDK components

---

### Question 4.2: What service components are needed?

**Core Components** (Always required):
- [x] API Layer (REST endpoints)
- [x] Authentication Middleware
- [x] Multi-Tenancy (Org/Project model)
- [x] Data Layer (database)

**For Observer Agents**:
- [ ] Ingestion Pipeline (batch processing)
- [ ] Time-series storage
- [ ] Aggregation engine
- [ ] Analytics/reporting

**For Controller Agents**:
- [ ] Validation endpoint
- [ ] Decision engine
- [ ] Policy storage
- [ ] Audit logging

**For Intelligent Agents** (Level 3-4):
- [ ] Risk scoring engine
- [ ] Behavioral analysis
- [ ] Anomaly detection
- [ ] Learning pipeline

**Output**: List of service components

---

## Step 5: Technology Stack

### Question 5.1: What runtime/language for SDK?

**Node.js SDK**:
- [x] TypeScript (strict mode)
- [x] Zero runtime dependencies for core
- [x] Peer dependencies: Express (optional)

**Python SDK**:
- [x] Python 3.10+
- [x] Type hints throughout
- [x] Optional: FastAPI integration

**Both**:
- [ ] Both Node.js and Python
- [ ] Start with one, add later

**Output**: SDK languages = [Node.js/Python/Both]

---

### Question 5.2: What tech stack for service?

**Service Framework**:
```
[ ] Node.js + Fastify (High performance)
    Pros: Fast, modern, TypeScript
    Cons: Smaller ecosystem than Express

[ ] Node.js + Express (Familiar)
    Pros: Huge ecosystem, well-known
    Cons: Slower than Fastify

[ ] Python + FastAPI (Async Python)
    Pros: Modern, async, great for ML
    Cons: Slightly slower than Node.js
```

**Database**:
```
[ ] PostgreSQL 15+
    - JSONB for flexible metadata
    - UUID for keys
    - Excellent for most use cases

[ ] PostgreSQL + TimescaleDB
    - For time-series data
    - Efficient aggregations
    → Use: High-volume time-based data

[ ] PostgreSQL + Redis
    - Redis for caching, rate limiting
    - PostgreSQL for persistent data
    → Use: Controller agents with caching
```

**Output**: Tech stack documented

---

## Step 6: Performance Requirements

### Question 6.1: What are your latency requirements?

**For Observer Agents**:
- Tracking latency: < [1ms/5ms/10ms] per event
- Flush latency: < [100ms/500ms/1s]

**For Controller Agents**:
- Validation latency: < [10ms/50ms/100ms]
- Decision latency: < [50ms/200ms/500ms]

**Critical**: Auth agents should target p95 < 50ms

**Output**: Latency targets documented

---

### Question 6.2: What throughput do you need?

**Expected volume**:
- Events/requests per second: [___]
- Peak vs average ratio: [2x/5x/10x]
- Growth projection: [+10%/+50%/+100% per year]

**Scaling strategy**:
- [ ] Vertical scaling sufficient (< 1000 RPS)
- [ ] Horizontal scaling needed (> 1000 RPS)
- [ ] Need auto-scaling

**Output**: Throughput requirements documented

---

## Step 7: Integration Design

### Question 7.1: How should projects integrate?

**For Observer Agents**:
```
[ ] Auto-instrumentation (Best DX)
    require('@myorg/agent/auto')
    → Zero code changes

[ ] Manual instrumentation
    agent.track(event)
    → More control

[ ] Middleware
    app.use(agent.middleware)
    → Express/FastAPI pattern
```

**For Controller Agents**:
```
[ ] Middleware (Required for blocking)
    app.get('/api/resource',
      agent.middleware.requireAuth(),
      handler
    )

[ ] Client library (For programmatic access)
    const user = await agent.validateToken(token)
```

**Output**: Integration pattern = [Auto/Manual/Middleware/Client]

---

### Question 7.2: What configuration is required?

**Minimal configuration** (recommended):
- API Key (for authentication)
- Project ID (for multi-tenancy)
- Optional: Service URL (defaults to production)

**Example**:
```typescript
const agent = new Agent({
  apiKey: process.env.AGENT_API_KEY,
  projectId: 'my-project'
  // Everything else has sensible defaults
});
```

**Principle**: < 5 minutes to integrate

**Output**: Configuration interface designed

---

## Step 8: Security & Compliance

### Question 8.1: What security measures are needed?

**Authentication**:
- [x] API Key authentication (required)
- [ ] OAuth/SSO (for user-facing)
- [ ] mTLS (for service-to-service)

**Authorization**:
- [x] Organization-level isolation
- [x] Project-level scoping
- [ ] Role-based access control (RBAC)

**Data Security**:
- [x] HTTPS only (enforced)
- [x] Secrets in environment variables
- [ ] PII scrubbing (if handling user data)
- [ ] Encryption at rest (if sensitive data)

**Audit**:
- [x] Audit logging (all auth events)
- [x] Immutable logs
- [x] 90-day retention minimum

**Output**: Security checklist completed

---

### Question 8.2: What compliance requirements?

Check all that apply:
- [ ] GDPR (EU data protection)
- [ ] CCPA (California privacy)
- [ ] SOC 2 (Security auditing)
- [ ] HIPAA (Healthcare data)
- [ ] PCI DSS (Payment data)

**Impact on design**:
- GDPR: Right to deletion, data export, consent
- SOC 2: Audit trails, encryption, access controls
- HIPAA: Encryption, access logs, BAA required

**Output**: Compliance requirements documented

---

## Step 9: Observability

### Question 9.1: What metrics should be tracked?

**Agent Health**:
- [x] Uptime / availability
- [x] Error rate
- [x] Latency (p50, p95, p99)
- [x] Throughput (requests/sec)

**Business Metrics** (agent-specific):
- [ ] Events tracked (for observers)
- [ ] Decisions made (for controllers)
- [ ] False positive rate (for intelligent agents)
- [ ] Coverage (% of ecosystem using agent)

**Output**: Metrics list documented

---

### Question 9.2: What alerting is needed?

**Critical Alerts** (Page on-call):
- Service down (< 99% uptime)
- Error rate > 5%
- Latency p95 > target * 2
- Security breach detected

**Warning Alerts** (Email/Slack):
- Error rate > 1%
- Latency p95 > target
- Capacity > 80%
- Anomaly detected

**Output**: Alerting rules defined

---

## Step 10: Validation

### Validation Checklist

Before implementation, validate your design:

**Autonomy** ✓
- [ ] Agent acts independently
- [ ] Self-registration implemented
- [ ] Graceful failure handling

**Performance** ✓
- [ ] Latency targets defined and achievable
- [ ] Throughput requirements specified
- [ ] Load testing plan created

**Resilience** ✓
- [ ] Retry logic with backoff
- [ ] Circuit breaker for failures
- [ ] Graceful degradation path
- [ ] Health checks exposed

**Security** ✓
- [ ] Authentication required
- [ ] Authorization implemented
- [ ] Secrets management plan
- [ ] Audit logging enabled

**Developer Experience** ✓
- [ ] < 5 minute integration time
- [ ] Minimal configuration required
- [ ] Clear error messages
- [ ] Documentation complete

**Observability** ✓
- [ ] Structured logging
- [ ] Metrics collection
- [ ] Health checks
- [ ] Alerting configured

---

## Decision Summary Template

Use this template to document your decisions:

```markdown
# [Agent Name] Design Decisions

## Problem Statement
[One sentence description]

## Classification
- Autonomy Level: [1-5]
- Operation Mode: [Observer/Controller/Hybrid]
- Intelligence Level: [0-4]

## Architecture
- Deployment: [SDK-Only/Service-Only/Hybrid]
- Communication: [Push/Pull/Request-Response/Mixed]
- Data Flow: [Sync/Async/Buffered]

## Components

### SDK
- Languages: [Node.js/Python/Both]
- Core: Config, Lifecycle, Communication
- Optional: [List components]

### Service
- Framework: [Fastify/Express/FastAPI]
- Database: [PostgreSQL/TimescaleDB]
- Cache: [Redis/None]
- Optional: [List components]

## Performance
- Latency Target: [X ms]
- Throughput Target: [X RPS]
- Scaling Strategy: [Vertical/Horizontal]

## Integration
- Pattern: [Auto/Manual/Middleware]
- Configuration: [List required fields]
- Integration Time: [< 5 minutes]

## Security
- Authentication: [API Key/OAuth/mTLS]
- Data Security: [List measures]
- Compliance: [GDPR/SOC2/etc.]

## Implementation Plan
- Sprint 1-2: MVP (core functionality)
- Sprint 3-4: Resilience
- Sprint 5-6: Intelligence (if applicable)
- Sprint 7-8: Production hardening
- Sprint 9+: Rollout

## Template Selection
Based on decisions above: [passive-observer/active-controller/hybrid]
```

---

## Example Walkthrough: Rate Limit Agent

Let's walk through designing a rate limit agent:

**Step 1: Problem**
- "Enforce consistent rate limits across all API endpoints in multiple projects"

**Step 2: Classification**
- Autonomy: Level 2 (Reactive - enforces rules)
- Mode: Controller (blocks requests)
- Intelligence: Level 1 (simple counting)

**Step 3: Architecture**
- Deployment: Hybrid (SDK + Service)
- Communication: Request/Response (sync)
- Data Flow: Synchronous

**Step 4: Components**
- SDK: Middleware, Cache (for distributed counting)
- Service: Rate limit store (Redis), policy management

**Step 5: Tech Stack**
- SDK: Node.js TypeScript
- Service: Node.js + Fastify + Redis

**Step 6: Performance**
- Latency: < 5ms (middleware must be fast!)
- Throughput: 10,000 RPS per instance

**Step 7: Integration**
- Middleware: `app.use(rateLimit.middleware())`
- Config: API key, rate limit rules

**Step 8: Security**
- API key auth
- Rate limits prevent DoS
- Audit logs for violations

**Template**: active-controller

---

## Next Steps

1. Use your decisions to select appropriate template:
   - `templates/passive-observer/` - For Level 2 observer agents
   - `templates/active-controller/` - For Level 2-4 controller agents
   - `templates/hybrid/` - For mixed patterns

2. Review patterns in [PATTERNS.md](PATTERNS.md)

3. Reference intelligence guide in [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md)

4. Follow design guide in [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md)

5. Start implementing!

---

## Common Decision Paths

### Path 1: Simple Observer (like costing-agent)
1. Problem: Track something across projects
2. Autonomy: Level 2, Observer, Intelligence 1
3. Architecture: Hybrid, Push, Buffered
4. Template: **passive-observer**

### Path 2: Intelligent Security (like auth-agent)
1. Problem: Security enforcement with learning
2. Autonomy: Level 4, Controller, Intelligence 4
3. Architecture: Hybrid, Request/Response, Sync
4. Template: **active-controller + intelligence**

### Path 3: Monitoring with Alerts
1. Problem: Detect performance issues
2. Autonomy: Level 3, Observer, Intelligence 2
3. Architecture: Hybrid, Push, Async
4. Template: **hybrid**

Happy agent building! 🚀
