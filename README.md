# Agent Framework

A comprehensive framework for designing and building intelligent, autonomous agents for your software ecosystem.

## What Is This Framework?

This framework provides:

1. **Design Methodology**: Step-by-step guide to design agents from concept to architecture
2. **Reusable Patterns**: Proven architectural patterns extracted from production agents
3. **Component Library**: Pre-built components for common agent capabilities
4. **Templates**: Starter templates for different agent types
5. **Decision Framework**: Decision trees to guide architectural choices

## Agent vs Service: Key Distinction

### Traditional Service
- Receives requests, returns responses
- Stateless or session-based
- Explicit invocation by clients
- Fixed behavior based on configuration
- No learning or adaptation

### Agent
- **Autonomous**: Acts independently without explicit invocation
- **Proactive**: Initiates actions based on observations
- **Adaptive**: Learns and adjusts behavior over time
- **Intelligent**: Makes decisions based on context and goals
- **Communicative**: Interacts with other agents and systems

## Agent Taxonomy

### By Autonomy Level

**Level 1: Instrumented** (e.g., logging, metrics)
- Passive observation only
- No decision-making
- Example: Simple telemetry

**Level 2: Reactive** (e.g., costing-agent)
- Automated data collection
- Self-registration
- Buffer management
- Example: Cost tracking with auto-instrumentation

**Level 3: Proactive** (e.g., monitoring agents)
- Pattern detection
- Anomaly alerting
- Automated reporting
- Example: Performance monitoring with alerts

**Level 4: Autonomous** (e.g., auth-agent)
- Independent decision-making
- Behavioral learning
- Autonomous threat response
- Example: Security agent that blocks attacks

**Level 5: Collaborative** (future)
- Multi-agent coordination
- Emergent behavior
- Collective intelligence
- Example: Swarm optimization

### By Purpose

**Observer Agents** (Passive)
- Monitor and report
- No system impact
- Async, non-blocking
- Examples: costing-agent, logging-agent, metrics-agent

**Controller Agents** (Active)
- Enforce policies
- Block/allow actions
- Sync, blocking
- Examples: auth-agent, rate-limit-agent, compliance-agent

**Optimizer Agents** (Adaptive)
- Improve performance
- Resource allocation
- Auto-scaling
- Examples: cache-agent, load-balancer-agent

**Coordinator Agents** (Orchestration)
- Workflow management
- Multi-service coordination
- Event choreography
- Examples: deployment-agent, workflow-agent

## Don't Build Agents for Everything

**Critical Principle: Agents are not the default solution.**

Use agents **only** for tasks that are **ambiguous, high-value, and verifiable**. If you can map out a deterministic decision tree, use a **workflow** instead.

### The Agent vs. Workflow Decision

```
Can you map the decision tree completely?
  ↓
YES → Use a Workflow
  • Known sequence of steps
  • Deterministic branching logic
  • Predictable outcomes
  • Example: CI/CD pipeline, form validation

NO → Consider an Agent
  • Exploratory problem space
  • Uncertain number of iterations
  • Context-dependent decisions
  • Example: Code refactoring, threat hunting
```

### The Three-Factor Evaluation

Before building an agent, evaluate:

#### 1. **Complexity** - Is it truly exploratory?
- ✅ **Agent territory**: "Optimize this API's performance" (unknown path)
- ❌ **Workflow territory**: "Deploy this service to staging" (known steps)

**Question**: Would a flowchart fully capture the solution path?
- If YES → Workflow
- If NO → Agent

#### 2. **Value** - Is it worth the cost?

Agents consume tokens ($$) and add complexity. Calculate ROI:

```typescript
// Simple ROI Framework
const agentROI = {
  costs: {
    development: 40,        // Hours to build
    llmTokens: 1000,        // $/month in API calls
    maintenance: 10,        // Hours/month
    infrastructure: 50      // $/month hosting
  },

  benefits: {
    timeSaved: 80,          // Hours/month saved
    errorsPrevented: 5,     // Incidents/month avoided
    incidentCost: 2000      // $/incident
  },

  // Monthly value
  monthlyValue: (80 * 50) + (5 * 2000) - 1000 - 50 - (10 * 50),
  // = $4000 + $10000 - $1050 - $500 = $12,450/month

  breakEven: 40 * 50 / 12450,  // = 0.16 months (~5 days)
};
```

**Use agents when:**
- Value delivered > Token cost + complexity cost
- Break-even is < 1 month for production agents
- Problem recurs frequently enough to justify automation

**Don't use agents when:**
- One-time task (just do it manually)
- Token costs exceed value delivered
- Simpler automation (cron job, script) would suffice

#### 3. **Error Cost** - Can mistakes be caught?

Agents make mistakes. Can you tolerate and recover from them?

```
Error Cost Assessment:

LOW (Safe for agents)
  • Mistakes are obvious (tests fail)
  • Easy to undo (rollback deploy)
  • Human review available
  • Example: Code suggestions, draft emails

MEDIUM (Agents with oversight)
  • Mistakes detectable but not instant
  • Recovery requires effort
  • Requires validation gates
  • Example: Database migrations, config changes

HIGH (Avoid agents or require approval)
  • Mistakes are costly or dangerous
  • Difficult/impossible to undo
  • Legal/compliance implications
  • Example: Data deletion, financial transactions

CRITICAL (Never use autonomous agents)
  • Safety-critical systems
  • Life/health impact
  • Irreversible consequences
  • Example: Medical decisions, industrial control
```

**Agent Safety Pattern:**
```typescript
if (errorCost === 'LOW') {
  agent.autonomous = true;
} else if (errorCost === 'MEDIUM') {
  agent.requireValidation = true;  // Tests must pass
} else if (errorCost === 'HIGH') {
  agent.requireHumanApproval = true;
} else {
  throw new Error('Use human decision-making');
}
```

### Decision Matrix

| Complexity | Value | Error Cost | Recommendation |
|------------|-------|------------|----------------|
| Low (mappable) | Any | Any | ❌ **Workflow** - Don't use agent |
| High (exploratory) | Low | Any | ❌ **Manual** - Not worth automation |
| High | High | Critical | ❌ **Human + Tools** - Too risky |
| High | High | Low-Medium | ✅ **Agent** - Good fit |
| High | High | High | ⚠️ **Agent + Approval** - Supervised agent |

### Examples

**Good Agent Use Cases:**
- ✅ Code refactoring (exploratory, high value, low error cost)
- ✅ Threat hunting (exploratory, high value, low-medium error cost)
- ✅ Test generation (exploratory, high value, very low error cost)
- ✅ Performance optimization (exploratory, high value, reversible)

**Bad Agent Use Cases:**
- ❌ User registration (deterministic workflow, not exploratory)
- ❌ Spell checking (low value, token cost not justified)
- ❌ Financial auditing (high error cost, requires human judgment)
- ❌ Medical diagnosis (critical error cost, safety-critical)

## When to Build an Agent

✅ **Build an agent when**:
- **Uncertain steps**: You don't know how many iterations are needed to achieve the goal
- **Autonomy required**: System must decide what to do next without human input
- **Feedback available**: Success/failure can be measured automatically (tests, metrics, checks)
- **Low error cost**: Mistakes are tolerable, reversible, or easy to verify
- **Adaptive behavior**: System should learn and improve from experience
- **Proactive actions**: Want actions based on observations, not just requests
- **Cross-cutting concerns**: Applies across multiple projects in your ecosystem

❌ **Use a traditional service/workflow when**:
- **Predictable steps**: Deterministic sequence of operations is known upfront
- **High error cost**: Mistakes are expensive, dangerous, or irreversible
- **No feedback mechanism**: Can't automatically verify success
- **Manual approval needed**: Human must validate each decision
- **Simple CRUD operations**: Just reading/writing data
- **Project-specific**: Functionality doesn't need to be shared
- **Stateless processing**: No learning or adaptation needed

## Quick Start Decision Tree

```
┌─────────────────────────────────────┐
│ What are you trying to solve?       │
└──────────────┬──────────────────────┘
               │
        ┌──────▼────────┐
        │ Does it need  │
        │  to observe   │───No──► Use traditional service
        │  and track?   │
        └──────┬────────┘
               │ Yes
        ┌──────▼────────┐
        │ Does it need  │
        │  to make      │───No──► Passive Observer Agent
        │ decisions?    │         (use passive-observer template)
        └──────┬────────┘
               │ Yes
        ┌──────▼────────┐
        │ Does it block │
        │ application   │───No──► Proactive Observer Agent
        │  flow?        │         (use hybrid template)
        └──────┬────────┘
               │ Yes
        ┌──────▼────────┐
        │ Does it need  │
        │ to learn from │───No──► Active Controller Agent
        │  behavior?    │         (use active-controller template)
        └──────┬────────┘
               │ Yes
               ▼
        Intelligent Controller Agent
        (use active-controller template
         + intelligence components)
```

## Real-World Examples

### Costing Agent (Level 2: Reactive Observer)
**Purpose**: Track LLM API costs across all projects

**Agent Characteristics**:
- **Autonomous**: Auto-instruments SDKs, self-registers
- **Proactive**: Automatically captures costs without manual tracking
- **Non-blocking**: Fire-and-forget event streaming
- **Distributed**: SDK in each app, central aggregation service

**Why It's an Agent**: Acts independently to collect data, manages its own lifecycle, adapts to different SDK versions

### Auth Agent (Level 4: Autonomous Controller)
**Purpose**: Intelligent authentication & security enforcement

**Agent Characteristics**:
- **Autonomous**: Blocks threats without human approval
- **Adaptive**: Learns user behavior patterns, adjusts policies
- **Intelligent**: Risk scoring, anomaly detection, context-aware decisions
- **Proactive**: Hunts for threats, auto-remediates issues

**Why It's an Agent**: Makes independent security decisions, learns over time, takes autonomous actions

## Framework Components

### 1. Design Guides
- [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md) - **Core agent loop, tools, verification, metrics**
- [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md) - Comprehensive design methodology
- [DECISION_TREE.md](DECISION_TREE.md) - Step-by-step decision framework
- [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) - Intelligence capabilities guide

### 2. Pattern Library
- [PATTERNS.md](PATTERNS.md) - Reusable architectural patterns
  - Lifecycle patterns
  - Communication patterns
  - Resilience patterns
  - Multi-tenancy patterns
  - Instrumentation patterns

### 3. Component Library
- `components/sdk/` - Client-side SDK components
  - Event buffer
  - Transmitter
  - Lifecycle manager
  - Instrumentation layer

- `components/service/` - Server-side service components
  - Registration endpoint
  - Authentication middleware
  - Ingestion pipeline
  - Multi-tenant data model

- `components/intelligence/` - Intelligence layer components
  - Risk scoring engine
  - Behavioral analysis
  - Anomaly detection
  - Decision engine

### 4. Templates
- `templates/passive-observer/` - For monitoring/tracking agents
- `templates/active-controller/` - For enforcement/security agents
- `templates/hybrid/` - Combined patterns

### 5. Examples
- Real integration examples from costing-agent and auth-agent
- Step-by-step tutorials for common agent types

## Architecture Patterns

All agents in this ecosystem follow these core patterns:

### 1. Hybrid Deployment (Service + SDK)
- **Central Service**: Single source of truth, aggregation, intelligence
- **Distributed SDKs**: Embedded in client apps, local optimization
- **Benefits**: Scalability, resilience, low latency

### 2. Multi-Tenancy
```
Organization (top-level isolation)
  └─ Projects (logical grouping)
      └─ Deployments/Instances (physical instances)
          └─ Data (scoped appropriately)
```

### 3. Resilient Communication
- Retry with exponential backoff
- Circuit breaker pattern
- Graceful degradation
- Timeout handling

### 4. Observable & Explainable
- Structured logging
- Metrics collection
- Audit trails
- Decision reasoning

## Getting Started

### Step 1: Define Your Agent

Answer these questions:
1. What problem does it solve?
2. What observations does it need to make?
3. What decisions does it make autonomously?
4. Does it need to learn/adapt?
5. Is it blocking or non-blocking?

### Step 2: Choose Your Template

Based on your answers:
- **Passive Observer**: No decisions, just tracking → `passive-observer/`
- **Active Controller**: Makes decisions, blocks flow → `active-controller/`
- **Hybrid**: Both observation and control → `hybrid/`

### Step 3: Select Components

Add intelligence components as needed:
- Risk scoring (for security agents)
- Behavioral analysis (for adaptive agents)
- Anomaly detection (for threat detection)
- Decision engine (for autonomous actions)

### Step 4: Implement & Test

Use the component library and patterns to build your agent.

### Step 5: Start Simple, Evolve Complexity

**Don't build Level 4 on day one!** Follow this incremental path:

```
Phase 1: Rule-Based (Level 2 - Reactive)
  ↓ Ship to production, gather data
Phase 2: Pattern Detection (Level 3 - Proactive)
  ↓ A/B test against rule-based
Phase 3: Autonomous (Level 4)
  ↓ Gradual rollout with human oversight
Phase 4: Optimize & Refine
```

**Week 1-2**: Build Level 2 (rule-based, deterministic logic)
- Simple if/then rules
- Fixed thresholds
- No ML required
- **Goal**: Ship quickly, validate concept

**Week 3-4**: Add Level 3 (pattern detection)
- Learn baselines
- Detect anomalies
- Dynamic thresholds
- **Goal**: Reduce false positives

**Week 5-8**: Add Level 4 (autonomous decisions)
- Risk scoring
- Multi-factor decisions
- Autonomous actions
- **Goal**: Maximize effectiveness

**Month 3+**: Optimize continuously
- Refine models
- Add LLM enhancement if needed
- Improve loop efficiency
- **Goal**: Reduce cost, improve accuracy

See [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md) for detailed guidance on incremental development.

## Design Principles

When building agents, follow these principles:

1. **Autonomy First**: Agent should act independently
2. **Resilience Built-In**: Handle failures gracefully
3. **Explainability Required**: Every decision must be traceable
4. **Privacy Conscious**: Protect sensitive data
5. **Performance Critical**: Optimize for low latency
6. **Security by Default**: Never compromise on security
7. **Observable Always**: Metrics, logs, traces for all actions

## Documentation Structure

```
agent-framework/
├── README.md                        # This file - Overview & getting started
├── FRAMEWORK_SUMMARY.md            # 📋 ONE-PAGE VISUAL SUMMARY
├── CORE_AGENT_MECHANICS.md         # Core loop, Model+Tools+Env, empathy, harnesses
├── AGENT_DESIGN_GUIDE.md           # 6-phase design methodology
├── DECISION_TREE.md                # Step-by-step architectural decisions
├── PATTERNS.md                     # Core Agent Loop + architectural patterns
├── MULTI_AGENT_PATTERNS.md         # Multi-agent coordination & safety
├── INTELLIGENCE_LAYERS.md          # Levels 0-4 intelligence capabilities
├── templates/                      # Starter templates
│   ├── passive-observer/
│   ├── active-controller/
│   └── hybrid/
├── components/                     # Reusable components
│   ├── sdk/
│   ├── service/
│   └── intelligence/
└── examples/                       # Real-world examples
    ├── costing-agent-walkthrough.md
    └── auth-agent-walkthrough.md
```

## Technology Stack Recommendations

### For Node.js Agents
- **Runtime**: Node.js 20+
- **Language**: TypeScript (strict mode)
- **Service Framework**: Fastify (performance) or Express (familiarity)
- **Database**: PostgreSQL + Redis
- **Testing**: Jest, Supertest

### For Python Agents
- **Runtime**: Python 3.10+
- **Framework**: FastAPI (async/await support)
- **Database**: PostgreSQL via SQLAlchemy
- **Testing**: pytest, pytest-asyncio

### Common Infrastructure
- **Database**: PostgreSQL 15+ (JSONB, UUID, time-series with TimescaleDB)
- **Cache**: Redis (sessions, rate limiting, distributed locks)
- **Queue**: Bull/BullMQ (Node.js) or Celery (Python)
- **Deployment**: Docker + Kubernetes

## Contributing New Agents

When you build a new agent:

1. Follow the design guide
2. Use existing patterns where possible
3. Document decisions (ADRs)
4. Add examples to this framework
5. Extract reusable components for others

## Examples of Agents to Build

### Security & Compliance
- **Auth Agent** ✅ (completed)
- **Rate Limit Agent**: Intelligent rate limiting across services
- **Compliance Agent**: Automated GDPR/SOC2 compliance checks
- **Audit Agent**: Comprehensive audit trail aggregation

### Operations & Performance
- **Costing Agent** ✅ (completed)
- **Cache Agent**: Intelligent cache warming and invalidation
- **Scaling Agent**: Auto-scaling based on patterns
- **Deployment Agent**: Canary deployments with auto-rollback

### Quality & Reliability
- **Testing Agent**: Automated testing based on code changes
- **Error Agent**: Intelligent error aggregation and triage
- **Performance Agent**: Automated performance regression detection
- **Availability Agent**: Circuit breaker coordination

### Business Intelligence
- **Analytics Agent**: Cross-project analytics aggregation
- **Recommendation Agent**: Intelligent feature recommendations
- **Experiment Agent**: A/B testing coordination
- **Reporting Agent**: Automated insight generation

## License

TBD

## References

- [Costing Agent](../costing-agent/)
- [Auth Agent](../auth-agent/)
- Agent-Oriented Software Engineering (AOSE)
- Microservices Architecture Patterns
- Multi-Agent Systems (MAS)
