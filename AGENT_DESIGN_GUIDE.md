# Agent Design Guide

A systematic methodology for designing intelligent, autonomous agents from concept to implementation.

## Design Process Overview

```
1. Problem Definition
   ↓
2. Agent Classification
   ↓
3. Architecture Selection
   ↓
4. Component Selection
   ↓
5. Interface Design
   ↓
6. Implementation Planning
```

---

## Phase 1: Problem Definition

### Questions to Answer

**1. What problem are you solving?**
- Describe in one sentence
- Example: "Track LLM API costs across all projects" (costing-agent)
- Example: "Provide intelligent authentication with autonomous threat response" (auth-agent)

**2. Why can't a traditional service solve this?**
- Need for autonomy?
- Need for learning/adaptation?
- Need for proactive behavior?
- Cross-cutting concern?

**3. Who are the stakeholders?**
- Developers (SDK users)
- Operators (dashboard users)
- End users (affected by agent)
- Security/compliance teams

**4. What are the success metrics?**
- Performance (latency, throughput)
- Accuracy (false positive/negative rates)
- Coverage (% of ecosystem using it)
- Value delivered (cost saved, threats blocked)

### Output: Problem Statement

Template:
```
Problem: [One sentence description]

Current State: [How is this handled today?]

Desired State: [What should the agent enable?]

Why Agent: [Why agent vs traditional service?]

Success Metrics:
- Metric 1: [target value]
- Metric 2: [target value]
```

Example (Costing Agent):
```
Problem: No visibility into LLM API costs across multiple projects

Current State: Manual tracking, inconsistent, no aggregation

Desired State: Automatic cost tracking with zero developer effort

Why Agent: Needs autonomous instrumentation, self-registration,
cross-project aggregation

Success Metrics:
- Integration time: < 5 minutes per project
- Coverage: 100% of LLM calls tracked
- Overhead: < 1ms per call
```

---

## Phase 2: Agent Classification

### Dimension 1: Autonomy Level

**Level 1: Instrumented** (Minimal Autonomy)
- Characteristics: Passive data collection, no decisions
- Example: Simple logging agent
- Template: Basic SDK with event emitter

**Level 2: Reactive** (Low Autonomy)
- Characteristics: Automated collection, self-management, buffering
- Example: Costing-agent
- Template: passive-observer

**Level 3: Proactive** (Medium Autonomy)
- Characteristics: Pattern detection, alerting, recommendations
- Example: Performance monitoring agent
- Template: hybrid (observer + alerting)

**Level 4: Autonomous** (High Autonomy)
- Characteristics: Independent decisions, behavioral learning, auto-remediation
- Example: Auth-agent
- Template: active-controller + intelligence

**Level 5: Collaborative** (Very High Autonomy)
- Characteristics: Multi-agent coordination, emergent behavior
- Example: Swarm deployment agent
- Template: Custom (not yet standardized)

### Dimension 2: Operation Mode

**Observer** (Non-Blocking)
- Agent observes but doesn't affect application flow
- Asynchronous operation
- Fire-and-forget communication
- Use case: Monitoring, tracking, analytics

**Controller** (Blocking)
- Agent makes decisions that affect application flow
- Synchronous operation
- Request/response communication
- Use case: Authentication, authorization, rate limiting

**Hybrid** (Both)
- Agent observes and controls based on context
- Mixed sync/async operations
- Use case: Adaptive systems, feedback loops

### Dimension 3: Intelligence Level

See [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) for details.

**Level 0: No Intelligence** - Pure instrumentation
**Level 1: Aggregation** - Statistical summaries
**Level 2: Pattern Detection** - Anomaly detection
**Level 3: Learning** - Behavioral models
**Level 4: Autonomous Decision-Making** - Risk-based actions

### Classification Matrix

```
                    Observer          Controller       Hybrid
                    (Non-blocking)    (Blocking)       (Both)
┌────────────────┬──────────────────────────────────────────┐
│ Level 1:       │  Logging          N/A              N/A   │
│ Instrumented   │  agent                                   │
├────────────────┼──────────────────────────────────────────┤
│ Level 2:       │  Costing          Rate-limit       N/A   │
│ Reactive       │  agent            agent                  │
├────────────────┼──────────────────────────────────────────┤
│ Level 3:       │  Metrics          Validation      Canary │
│ Proactive      │  agent            agent           deploy │
├────────────────┼──────────────────────────────────────────┤
│ Level 4:       │  Analytics        Auth-agent      Cache  │
│ Autonomous     │  agent                            agent  │
├────────────────┼──────────────────────────────────────────┤
│ Level 5:       │  Insight          Security        Full   │
│ Collaborative  │  agent            mesh            mesh   │
└────────────────┴──────────────────────────────────────────┘
```

### Output: Agent Profile

Template:
```
Agent Name: [name]-agent

Classification:
- Autonomy Level: [1-5]
- Operation Mode: [Observer/Controller/Hybrid]
- Intelligence Level: [0-4]

Characteristics:
- Autonomous: [Yes/No - what behaviors?]
- Proactive: [Yes/No - what triggers?]
- Adaptive: [Yes/No - what learning?]
- Blocking: [Yes/No - when blocks?]
```

---

## Phase 3: Architecture Selection

### Decision: Deployment Model

**Option A: SDK-Only**
- Pros: No infrastructure, simplest
- Cons: No central visibility, no cross-project intelligence
- Use when: Single-project, no aggregation needed

**Option B: Service-Only**
- Pros: Centralized control
- Cons: No local optimization, higher latency
- Use when: Centralized decision-making only

**Option C: Hybrid (Service + SDK)** ⭐ Recommended
- Pros: Best of both worlds
- Cons: More complexity
- Use when: Need both local and central capabilities

### Decision: Communication Pattern

**Push Model** (SDK → Service)
- Agent pushes events to service
- Use for: Observer agents, event streaming
- Example: Costing-agent

**Pull Model** (Service → SDK)
- Service queries agent
- Use for: Status checks, health monitoring
- Example: Deployment health agent

**Request/Response** (Bidirectional)
- Agent sends request, waits for response
- Use for: Controller agents, validation
- Example: Auth-agent

**Hybrid**
- Push for events, pull for validation
- Use for: Complex agents
- Example: Combined monitoring + control

### Decision: Data Flow

**Synchronous**
- Application waits for agent response
- Required for: Controllers (auth, rate-limit)
- Latency critical: < 50ms

**Asynchronous**
- Application continues immediately
- Required for: Observers (logging, metrics)
- Latency flexible: < 1s

**Buffered**
- Agent batches before sending
- Efficient for high-volume data
- Example: Event tracking

### Output: Architecture Design

Template:
```
Deployment Model: [SDK-Only/Service-Only/Hybrid]

Components:
- SDK: [Yes/No - languages: Node.js, Python, etc.]
- Service: [Yes/No - tech stack: Node.js/Fastify, etc.]
- Database: [PostgreSQL, Redis, TimescaleDB, etc.]

Communication:
- Pattern: [Push/Pull/Request-Response/Hybrid]
- Protocol: [HTTPS/REST, gRPC, WebSocket, etc.]
- Data Flow: [Sync/Async/Buffered]

Integration Points:
- How projects integrate: [Auto-instrumentation, middleware, etc.]
- Configuration: [API key, project ID, etc.]
- Minimal config: [True/False]
```

---

## Phase 3.5: Tool Design & Verification

Before selecting components, define how your agent will **act** (tools) and **verify success** (verification mechanisms).

### Tool Design

Tools are the agent's capabilities - functions, APIs, actions it can take to pursue goals.

**Tool Specification Template:**

```typescript
interface AgentTool {
  // Identity
  name: string;
  description: string;

  // Interface
  parameters: Record<string, ParameterSpec>;
  returnType: TypeSpec;

  // Safety
  sideEffects: 'none' | 'read-only' | 'mutating' | 'dangerous';
  errorCost: 'low' | 'medium' | 'high' | 'critical';
  idempotent: boolean;
  requiresApproval: boolean;

  // Constraints
  rateLimit?: { requests: number; window: string };
  permissions?: string[];

  // Verification
  successCriteria: string;
  failureConditions: string[];
}
```

**Example: Block User Tool (Auth Agent)**

```typescript
const blockUserTool = {
  name: "blockUser",
  description: "Temporarily block a user from authentication",

  parameters: {
    userId: { type: "string", required: true },
    reason: { type: "string", required: true },
    durationMinutes: { type: "number", default: 60, max: 1440 }
  },

  sideEffects: "dangerous", // Affects user access
  errorCost: "high",         // False positive blocks legitimate users
  idempotent: true,          // Safe to call twice
  requiresApproval: false,   // Autonomous for high-confidence

  successCriteria: "User cannot authenticate until expiry",
  failureConditions: ["User is admin", "User already blocked"]
};
```

**Tool Design Checklist:**

- [ ] Clear interface with explicit parameters
- [ ] Documented side effects and error cost
- [ ] Idempotent where possible (safe to retry)
- [ ] Rate limits and permissions defined
- [ ] Fast feedback (< 5s preferred)
- [ ] Success/failure signals are clear
- [ ] Composable with other tools

### Verification Mechanisms

How does your agent know it succeeded?

**Automated Verification (Preferred):**

1. **Unit Tests**
   ```typescript
   // Goal: "Make user registration test pass"
   const result = await runTest('user-registration.test.ts');
   const verified = result.exitCode === 0;
   ```

2. **Metric Thresholds**
   ```typescript
   // Goal: "Deploy with < 0.1% error rate"
   const metrics = await getMetrics();
   const verified = metrics.errorRate < 0.001;
   ```

3. **Property Checks**
   ```typescript
   // Goal: "Improve cache hit rate"
   const before = await getCacheHitRate();
   await agent.optimize();
   const after = await getCacheHitRate();
   const verified = after > before;
   ```

4. **External Oracles**
   ```typescript
   // Goal: "Block all known malicious IPs"
   const threats = await getThreatIntel();
   const blocked = await getBlockedIPs();
   const verified = threats.every(ip => blocked.includes(ip));
   ```

**Human Verification (When Necessary):**

1. **Human-in-the-Loop**
   - High-stakes actions (data deletion, production changes)
   - Low confidence decisions (< 70%)
   - Required for dangerous side effects

2. **Confidence Thresholds**
   ```typescript
   if (decision.confidence > 0.95) {
     await execute();  // Autonomous
   } else if (decision.confidence > 0.70) {
     await requestReview(decision);  // Ask human
   } else {
     await skipAction();  // Confidence too low
   }
   ```

**Verification Checklist:**

- [ ] Success criteria are measurable
- [ ] Automated checks where possible
- [ ] Human approval for high-risk actions
- [ ] Fast feedback loop (< 5s for tests)
- [ ] Clear pass/fail signals
- [ ] Actionable error messages

### Agent Loop Integration

Your tools and verification integrate into the agent loop:

```typescript
// GOAL: Define success criteria
const goal = {
  description: "Stop brute force attack",
  successCriteria: (obs) => obs.failedLogins < 5 && obs.attackerBlocked,
  maxIterations: 5
};

// PLAN: Choose which tool to use
const plan = await this.plan(goal, currentState);

// ACT: Execute the tool
const result = await this.tools[plan.toolName].execute(plan.params);

// OBSERVE: Collect feedback
const observation = await this.observe(result);

// REFLECT: Verify goal achieved
const goalAchieved = goal.successCriteria(observation);
```

See [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md) for comprehensive guidance on tools, verification, and the agent loop.

---

## Phase 4: Component Selection

### Core Components (Required)

**For SDK:**
1. **Configuration Manager**
   - Load from env vars
   - Provide sensible defaults
   - Validate configuration

2. **Lifecycle Manager**
   - init() - Setup and registration
   - shutdown() - Graceful cleanup
   - Health check

3. **Communication Layer**
   - HTTP client (for service communication)
   - Retry logic
   - Error handling

**For Service:**
1. **API Layer**
   - REST endpoints
   - Authentication middleware
   - Rate limiting

2. **Data Layer**
   - Database models
   - Migration system
   - Query optimization

3. **Multi-Tenancy**
   - Organization model
   - Project model
   - Access control

### Optional Components

**For Observer Agents:**
- Event Buffer (batching)
- Transmitter (resilient sending)
- Instrumentation Layer (auto-capture)

**For Controller Agents:**
- Validation Layer (request validation)
- Decision Engine (policy enforcement)
- Cache Layer (performance)

**For Intelligent Agents:**
- Risk Scoring Engine
- Behavioral Analysis
- Anomaly Detection
- Learning Loop

See [PATTERNS.md](PATTERNS.md) for implementation details.

### Output: Component List

Template:
```
SDK Components:
- [ ] Configuration Manager
- [ ] Lifecycle Manager
- [ ] Communication Layer
- [ ] Event Buffer (if observer)
- [ ] Validation Cache (if controller)
- [ ] Instrumentation (if auto-capture)

Service Components:
- [ ] API Layer (REST/gRPC)
- [ ] Authentication Middleware
- [ ] Multi-Tenancy (Org/Project model)
- [ ] Data Layer (PostgreSQL/Redis)
- [ ] Intelligence Layer (if adaptive)

Infrastructure:
- [ ] Database: [PostgreSQL/etc.]
- [ ] Cache: [Redis/etc.]
- [ ] Queue: [Bull/etc.]
```

---

## Phase 5: Interface Design

### SDK Interface

**For Observer Agents:**
```typescript
class ObserverAgent {
  constructor(config: Config)

  // Track events
  track(event: Event): void

  // Force sync
  flush(): Promise<void>

  // Lifecycle
  shutdown(): Promise<void>
}
```

**For Controller Agents:**
```typescript
class ControllerAgent {
  constructor(config: Config)

  // Middleware (Express)
  middleware: {
    requireAuth(): Middleware
    optionalAuth(): Middleware
  }

  // Client methods
  client: {
    validate(token: string): Promise<User>
    revoke(token: string): Promise<void>
  }
}
```

### API Interface

**Registration Endpoint:**
```
POST /api/v1/register
Body: { project_id, deployment_info, metadata }
Response: { deployment_id, config }
```

**Tracking Endpoint (Observer):**
```
POST /api/v1/track
Body: { events: [...] }
Response: { success: true }
```

**Validation Endpoint (Controller):**
```
POST /api/v1/validate
Body: { token, context }
Response: { valid: true, user, risk_score }
```

### Output: API Specification

Use OpenAPI 3.0 format for formal specification.

---

## Phase 6: Implementation Planning

### Implementation Sequence

**Phase 1: MVP (Core Functionality)**
1. Basic SDK with config and lifecycle
2. Central service with registration
3. Single strategy/capability
4. Basic testing

**Phase 2: Resilience**
1. Retry logic and circuit breaker
2. Graceful degradation
3. Error handling and logging
4. Integration testing

**Phase 3: Intelligence (if applicable)**
1. Data collection for learning
2. Pattern detection
3. Decision engine
4. Feedback loop

**Phase 4: Production Hardening**
1. Comprehensive testing (unit, integration, e2e)
2. Performance optimization
3. Security audit
4. Documentation

**Phase 5: Ecosystem Integration**
1. Integrate with existing projects
2. Operator dashboard
3. Monitoring and alerting
4. Runbooks and playbooks

### Testing Strategy

**Unit Tests:**
- All business logic
- Edge cases
- Error conditions

**Integration Tests:**
- SDK ↔ Service communication
- Database interactions
- Authentication flows

**End-to-End Tests:**
- Full user workflows
- Multi-project scenarios
- Failure scenarios

**Performance Tests:**
- Load testing (target RPS)
- Latency testing (p95, p99)
- Stress testing (failure modes)

**Security Tests:**
- Authentication bypass attempts
- Injection attacks (SQL, XSS)
- Rate limit evasion
- Secret leakage

### Output: Implementation Roadmap

Template:
```
Sprint 1-2: MVP
- [ ] SDK core (config, lifecycle)
- [ ] Service core (API, database)
- [ ] Basic integration
- [ ] Unit tests

Sprint 3-4: Resilience
- [ ] Retry and circuit breaker
- [ ] Error handling
- [ ] Integration tests
- [ ] Documentation

Sprint 5-6: Intelligence (if applicable)
- [ ] Data collection
- [ ] Analysis engine
- [ ] Decision logic
- [ ] Feedback loop

Sprint 7-8: Production Ready
- [ ] Performance optimization
- [ ] Security audit
- [ ] Comprehensive testing
- [ ] Operator tools

Sprint 9+: Rollout
- [ ] Pilot with 1-2 projects
- [ ] Monitor and iterate
- [ ] Full ecosystem rollout
- [ ] Runbooks and training
```

---

## Design Validation Checklist

Before implementation, validate your design:

### Autonomy
- [ ] Agent acts independently without manual triggers
- [ ] Self-registration and lifecycle management
- [ ] Graceful handling of failures

### Performance
- [ ] Latency targets defined and achievable
- [ ] Throughput requirements specified
- [ ] Resource usage is reasonable

### Resilience
- [ ] Retry logic with backoff
- [ ] Circuit breaker for cascading failures
- [ ] Graceful degradation path

### Security
- [ ] Authentication required
- [ ] Authorization implemented
- [ ] Secrets management plan
- [ ] Audit logging

### Observability
- [ ] Structured logging
- [ ] Metrics collection
- [ ] Distributed tracing
- [ ] Health checks

### Privacy
- [ ] PII handling defined
- [ ] Data retention policy
- [ ] GDPR/compliance consideration
- [ ] User consent if needed

### Developer Experience
- [ ] < 5 minute integration time
- [ ] Minimal configuration
- [ ] Clear error messages
- [ ] Comprehensive documentation

---

## Common Pitfalls

### 1. Over-Engineering
**Problem**: Adding too much intelligence/autonomy upfront
**Solution**: Start with Level 2 (Reactive), add intelligence incrementally

### 2. Under-Engineering Resilience
**Problem**: Not handling failures gracefully
**Solution**: Build retry, circuit breaker, graceful degradation from day 1

### 3. Ignoring Performance
**Problem**: High latency breaks user experience
**Solution**: Define latency budgets early, measure continuously

### 4. Poor Observability
**Problem**: Can't debug issues in production
**Solution**: Structured logging, metrics, traces from start

### 5. Weak Security
**Problem**: Agents have powerful access, become attack vectors
**Solution**: Security audit before production, principle of least privilege

---

## Example Walkthrough: Designing a Cache Agent

**Problem**: Inefficient caching across projects, manual cache warming

**Classification**:
- Autonomy: Level 4 (Autonomous)
- Mode: Hybrid (observes access patterns, controls cache)
- Intelligence: Level 3 (learns patterns, predicts needs)

**Architecture**:
- Deployment: Hybrid (SDK + Service)
- Communication: Push (metrics) + Pull (cache validation)
- Data Flow: Async for metrics, sync for cache hits

**Components**:
- SDK: Instrumentation layer, cache client
- Service: Pattern analysis, predictive warming, cache coordination
- Intelligence: Access pattern learning, eviction predictor

**Interface**:
```typescript
// SDK
cache.get(key)  // Instrumented, reports hits/misses
cache.set(key, value, ttl)

// Agent learns and auto-warms based on patterns
```

**Implementation**: Start with basic instrumentation (Sprint 1-2), add pattern learning (Sprint 3-4), add predictive warming (Sprint 5-6)

---

## Next Steps

1. Use [DECISION_TREE.md](DECISION_TREE.md) for step-by-step guidance
2. Review [PATTERNS.md](PATTERNS.md) for reusable patterns
3. Select appropriate template from `templates/`
4. Reference [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) if needed

Happy agent building!
