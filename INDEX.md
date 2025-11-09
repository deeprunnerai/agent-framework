# Agent Framework - Complete Index

Quick navigation to all framework resources, organized by use case.

---

## 🚀 New to Agents? Start Here

1. **[QUICK_START.md](QUICK_START.md)** ⭐ START HERE
   - How to identify if you need an agent
   - Step-by-step guide to building one
   - Real-world example walkthrough

2. **[README.md](README.md)**
   - Framework overview
   - Agent vs. Service distinction
   - When to build agents (and when NOT to)

3. **[FRAMEWORK_SUMMARY.md](FRAMEWORK_SUMMARY.md)**
   - One-page visual reference
   - Core principles
   - Quick checklist

---

## 📖 Core Concepts

### Understanding Agents

**[CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md)** (Comprehensive - Read this!)
- **Agent Composition**: Model + Tools + Environment
- **The Agent Loop**: Goal → Plan → Act → Observe → Reflect
- **Tool Design**: How to equip agents with capabilities
- **Verification**: How agents know they succeeded
- **Agent Empathy**: Debugging from agent's perspective
- **Evaluation Harnesses**: Testing agent behavior
- **Prompt Engineering**: For LLM-enhanced agents (future)

**Key Sections**:
- [Agent Composition](CORE_AGENT_MECHANICS.md#0-agent-composition-model--tools--environment) - Lines 9-215
- [The Core Agent Loop](CORE_AGENT_MECHANICS.md#1-the-core-agent-loop) - Lines 217-273
- [Tool Design Principles](CORE_AGENT_MECHANICS.md#3-tool-design-principles) - Lines 412-514
- [Agent Empathy](CORE_AGENT_MECHANICS.md#7-agent-empathy-think-like-your-agent) - Lines 906-1184
- [Evaluation Harnesses](CORE_AGENT_MECHANICS.md#8-evaluation-harnesses) - Lines 1186-1408

### When to Build Agents

**[README.md](README.md#dont-build-agents-for-everything)** - Lines 90-251
- Agent vs. Workflow decision
- Three-Factor Evaluation (Complexity, Value, Error Cost)
- ROI calculation framework
- Decision matrix
- Good vs. bad use cases

---

## 🎯 Design & Planning

### Systematic Design Process

**[AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md)**
6-phase methodology:
1. Problem Definition - Lines 23-82
2. Agent Classification - Lines 84-184
3. Architecture Selection - Lines 186-266
4. Tool Design & Verification - Lines 268-428
5. Component Selection - Lines 430-510
6. Implementation Planning - Lines 580-677

### Step-by-Step Decisions

**[DECISION_TREE.md](DECISION_TREE.md)**
10 structured decision steps:
1. Problem Definition - Lines 13-27
2. Agent Classification - Lines 48-151
3. Architecture Selection - Lines 153-234
4. Component Selection - Lines 236-291
5. Technology Stack - Lines 293-352
6. Performance Requirements - Lines 354-386
7. Integration Design - Lines 388-443
8. Security & Compliance - Lines 445-491
9. Observability - Lines 493-529
10. Validation - Lines 531-571

**Includes**: Decision summary template, validation checklist, example walkthrough

---

## 🏗️ Implementation

### Reusable Patterns

**[PATTERNS.md](PATTERNS.md)**
15 production-tested patterns:

**Core**:
- Pattern 0: Core Agent Loop - Lines 17-243 ⭐ FUNDAMENTAL

**Lifecycle**:
- Pattern 1: Auto-Registration - Lines 248-289
- Pattern 2: Graceful Shutdown - Lines 295-342
- Pattern 3: Heartbeat - Lines 347-379

**Communication**:
- Pattern 4: Event Buffer - Lines 385-438
- Pattern 5: Resilient Transmitter - Lines 444-516
- Pattern 6: Request Compression - Lines 521-554

**Resilience**:
- Pattern 7: Graceful Degradation - Lines 559-597
- Pattern 8: Health Checks - Lines 602-642

**Instrumentation**:
- Pattern 9: Auto-Instrumentation - Lines 647-713
- Pattern 10: Middleware (Express) - Lines 718-787
- Pattern 11: Dependency Injection (FastAPI) - Lines 792-835

**Multi-Tenancy**:
- Pattern 12: Hierarchical Tenancy - Lines 840-889

**Caching**:
- Pattern 13: Local Token Validation - Lines 894-954

**Intelligence**:
- Pattern 14: Behavioral Profile - Lines 959-1017
- Pattern 15: Risk Scoring - Lines 1021-1077

### Templates

**[templates/passive-observer/README.md](templates/passive-observer/README.md)**
- For Level 2 Observer Agents
- Use when: non-blocking, fire-and-forget
- Example: costing-agent, metrics-agent
- Performance targets: < 1ms overhead, 10k events/sec

**[templates/active-controller/README.md](templates/active-controller/README.md)**
- For Level 2-4 Controller Agents
- Use when: blocking, decision-making required
- Example: auth-agent, rate-limit-agent
- Performance targets: p95 < 50ms, 10k req/sec, 99.9% uptime

**[templates/hybrid/README.md](templates/hybrid/README.md)**
- For mixed observer + controller patterns
- Use when: both observation and control needed

---

## 🧠 Intelligence

### Intelligence Levels

**[INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md)**

**Level 0: Pure Instrumentation** - Lines 22-58
- Collects raw data only
- Example: Simple logging

**Level 1: Aggregation & Analysis** - Lines 60-123
- Statistical aggregation, cost calculation
- Example: costing-agent

**Level 2: Pattern Detection** - Lines 125-193
- Anomaly detection, threshold alerts
- Example: performance monitoring

**Level 3: Behavioral Learning** - Lines 195-307
- Learns patterns, builds profiles, adapts
- Example: auth-agent (behavioral analysis)

**Level 4: Autonomous Decision-Making** - Lines 309-531
- Risk scoring, autonomous actions, self-improvement
- Example: auth-agent (full intelligence)

**Key Sections**:
- [Choosing Intelligence Level](INTELLIGENCE_LAYERS.md#choosing-intelligence-level) - Lines 549-595
- [Incremental Intelligence](INTELLIGENCE_LAYERS.md#incremental-intelligence) - Lines 634-655

---

## 🤝 Multi-Agent Systems

**[MULTI_AGENT_PATTERNS.md](MULTI_AGENT_PATTERNS.md)**

**When to Introduce a Second Agent** - Lines 17-143
- Trigger 1: Conflicting optimization goals
- Trigger 2: Distinct expertise domains
- Trigger 3: Different operational rhythms
- Trigger 4: Independent failure domains
- Decision tree: Should you add a second agent?

**Communication Patterns** - Lines 147-397
- Pattern 1: Fire-and-Forget Events
- Pattern 2: Request-Response RPC
- Pattern 3: Shared State (Coordination)
- Pattern 4: Hierarchical (Orchestrator + Workers)
- Pattern 5: Peer-to-Peer (Collaborative)

**Self-Improving Tools** - Lines 401-543
- Tools with feedback loops
- Learning from outcomes

**Safety Considerations** - Lines 547-648
- Agent isolation
- Conflict resolution
- Rate limiting
- Circuit breakers
- Timeouts

**Anti-Patterns** - Lines 707-741
- Premature multi-agent
- Deep agent chains
- Chatty agents
- No clear boundaries

---

## 📊 Operational Guide

### Metrics & Monitoring

**From [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#6-operational-metrics)**:

**Traditional Metrics**:
- Latency (p50, p95, p99)
- Throughput (req/sec)
- Error rate
- Availability

**Agent-Specific Metrics**:
- Avg iterations to goal (target: < 5)
- Convergence rate (target: > 95%)
- True positive rate (target: > 99%)
- False positive rate (target: < 1%)
- Cost per goal

### Testing

**Evaluation Harnesses** from [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#8-evaluation-harnesses):
- Test scenarios for agent behavior
- Pass rate tracking
- Integration with CI/CD
- Regression detection

### Performance Targets

**Observer Agents**:
- Tracking overhead: < 1ms per event
- Flush latency: < 500ms
- Memory: < 50MB for SDK
- Throughput: 10,000 events/sec

**Controller Agents** (Critical - in hot path):
- Validation: p95 < 50ms
- Decision: p95 < 200ms
- Cache hit rate: > 95%
- Throughput: 10,000 req/sec
- Availability: 99.9%+

---

## 🔐 Security

### Security Best Practices

**From [DECISION_TREE.md](DECISION_TREE.md#security--compliance)**:
- Authentication: API keys required
- Authorization: Org-level isolation
- Data security: HTTPS only, secrets in env vars
- Audit: Immutable logs, 90-day retention
- Fail-closed by default (for security agents)

**For Controller Agents** from [templates/active-controller/](templates/active-controller/README.md):
- ✅ Authentication required
- ✅ Audit logging (immutable)
- ✅ Secrets never logged
- ✅ Fail-closed by default
- ✅ Rate limiting on validation
- ✅ Input validation (prevent injection)

---

## 📚 Learning Paths

### 1. Quick Start (1 hour)
```
1. QUICK_START.md (30 min)
2. README.md (20 min)
3. FRAMEWORK_SUMMARY.md (10 min)
```

### 2. Design Fundamentals (4 hours)
```
1. CORE_AGENT_MECHANICS.md (2 hours)
   - Model + Tools + Environment
   - Agent Loop
   - Agent Empathy

2. AGENT_DESIGN_GUIDE.md (1 hour)
   - 6-phase methodology

3. DECISION_TREE.md (1 hour)
   - Work through for YOUR agent
```

### 3. Implementation (4 hours)
```
1. PATTERNS.md (2 hours)
   - Study patterns for your agent type
   - Copy implementations

2. Template README (1 hour)
   - Passive observer or active controller

3. INTELLIGENCE_LAYERS.md (1 hour)
   - Choose intelligence level
```

### 4. Advanced (3 hours)
```
1. MULTI_AGENT_PATTERNS.md (2 hours)
   - When to add second agent
   - Communication patterns

2. Evaluation & Testing (1 hour)
   - Build harnesses
   - Track metrics
```

**Total**: ~12 hours from zero to production-ready agent.

---

## 🎯 By Use Case

### "I want to track something across projects"
→ Start: [QUICK_START.md](QUICK_START.md)
→ Template: [passive-observer](templates/passive-observer/)
→ Patterns: Auto-Registration, Event Buffer, Resilient Transmitter
→ Example: Costing agent walkthrough in QUICK_START.md

### "I want to enforce a policy"
→ Start: [QUICK_START.md](QUICK_START.md)
→ Template: [active-controller](templates/active-controller/)
→ Patterns: Middleware, Local Validation, Fail-Closed
→ Security: [DECISION_TREE.md](DECISION_TREE.md#security--compliance)

### "I want intelligent security/fraud detection"
→ Start: [README.md](README.md#when-to-build-an-agent)
→ Intelligence: [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) Level 3-4
→ Patterns: Behavioral Profile, Risk Scoring
→ Incrementally: Start Level 2 → 3 → 4

### "I want multiple agents to work together"
→ Start: [MULTI_AGENT_PATTERNS.md](MULTI_AGENT_PATTERNS.md)
→ Decision: When to Introduce a Second Agent (Lines 17-143)
→ Warning: Most problems don't need multiple agents!

### "I'm stuck debugging my agent"
→ Read: [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#7-agent-empathy-think-like-your-agent)
→ Checklist: Debugging from agent's perspective
→ Trace: Loop tracing (Lines 1620-1686)

---

## 📋 Checklists

### Before Building
From [README.md](README.md#when-to-build-an-agent):
```
✓ Can't map decision tree completely (exploratory)
✓ Value > Token cost + Complexity cost
✓ Break-even < 1 month
✓ Error cost is tolerable
✓ Feedback is measurable
✓ Need autonomous behavior
```

### During Design
From [DECISION_TREE.md](DECISION_TREE.md#validation-checklist):
```
✓ Agent acts independently
✓ Latency targets defined
✓ Retry + circuit breaker included
✓ Authentication required
✓ < 5 minute integration
✓ Structured logging
```

### Before Shipping
From [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#summary):
```
✓ Loop is explicit with exit conditions
✓ Tools are well-designed (idempotent, fast)
✓ Verification built-in
✓ Started simple (Level 2)
✓ Metrics tracked (loop efficiency, accuracy)
✓ Evaluation harness with 3+ scenarios
```

---

## 🆘 Troubleshooting

### Common Issues

**"Should this be an agent?"**
→ Answer three-factor test: [README.md](README.md#the-three-factor-evaluation)

**"Which template should I use?"**
→ Decision tree: [QUICK_START.md](QUICK_START.md#phase-1-design-your-agent-use-the-framework)

**"Agent is stuck in infinite loop"**
→ Add maxIterations: [PATTERNS.md](PATTERNS.md#core-agent-loop-pattern)

**"Agent makes wrong decisions"**
→ Debug with empathy: [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#7-agent-empathy-think-like-your-agent)

**"False positive rate too high"**
→ Add behavioral learning: [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md#level-3-behavioral-learning)

**"Should I add a second agent?"**
→ Probably not: [MULTI_AGENT_PATTERNS.md](MULTI_AGENT_PATTERNS.md#when-to-introduce-a-second-agent)

---

## 📞 Getting Help

1. **Check the index above** - Find the right document
2. **Read QUICK_START.md** - Most questions answered there
3. **Search for keywords** - Use Ctrl+F in docs
4. **Follow decision trees** - Structured thinking helps

---

## 🗺️ Document Map

```
agent-framework/
├── INDEX.md ⭐ THIS FILE - Navigation hub
├── QUICK_START.md ⭐ START HERE - Identification & creation guide
├── README.md - Overview, when to use agents
├── FRAMEWORK_SUMMARY.md - One-page visual reference
│
├── CORE_AGENT_MECHANICS.md ⭐ ESSENTIAL - Loop, empathy, tools
├── AGENT_DESIGN_GUIDE.md - 6-phase methodology
├── DECISION_TREE.md - 10-step decision framework
│
├── PATTERNS.md - 15 reusable patterns with code
├── INTELLIGENCE_LAYERS.md - Level 0-4 intelligence
├── MULTI_AGENT_PATTERNS.md - Multi-agent systems
│
├── templates/
│   ├── passive-observer/ - For observer agents
│   ├── active-controller/ - For controller agents
│   └── hybrid/ - For mixed patterns
│
├── components/ (empty - implementations needed)
│   ├── sdk/
│   ├── service/
│   └── intelligence/
│
└── examples/ (empty - examples needed)
```

---

**Pro Tip**: Bookmark this INDEX.md for quick navigation!
