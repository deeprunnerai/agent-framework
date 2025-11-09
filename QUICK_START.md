# Quick Start: Identifying & Creating Agents

This guide shows you **exactly how to use this framework** to identify when you need an agent and how to build one.

---

## 🎯 How This Framework Helps You

### 1. **Agent Identification** (Prevent Building the Wrong Thing)

The framework provides a **rigorous decision system** to determine if you should build an agent:

#### Step 1: The Three-Factor Test

Ask these questions from [README.md](README.md#the-three-factor-evaluation):

**Factor 1: COMPLEXITY** - Is it exploratory?
```
❓ Can you map out all the steps in a flowchart?
   → YES: Use a workflow, NOT an agent
   → NO: Agent might be appropriate

Example:
✅ Agent: "Optimize API performance" (unknown path, need to try different approaches)
❌ Workflow: "Deploy service to staging" (known: build → test → deploy)
```

**Factor 2: VALUE** - Is it worth the cost?
```
❓ Calculate ROI using the formula in README.md:

Monthly Value = (Time Saved × $50/hr) + (Errors Prevented × $Cost/Error)
                - LLM Costs - Infrastructure - Maintenance

Break-even < 1 month? → Worth building
Break-even > 3 months? → Probably not worth it
```

**Factor 3: ERROR COST** - Can mistakes be caught?
```
❓ If the agent makes a mistake, what happens?

LOW (Safe for agents):
  • Mistakes obvious (tests fail)
  • Easy to undo (rollback)
  • Example: Code suggestions

HIGH (Avoid agents):
  • Mistakes costly (data loss)
  • Hard to undo (production outage)
  • Example: Database migrations

CRITICAL (Never use agents):
  • Safety-critical
  • Irreversible
  • Example: Medical decisions
```

**Decision Matrix**: [README.md](README.md#decision-matrix)
```
Complexity | Value | Error Cost | Decision
-----------|-------|------------|----------
High       | High  | Low-Medium | ✅ BUILD AGENT
High       | High  | High       | ⚠️ AGENT + APPROVAL
High       | Low   | Any        | ❌ MANUAL
Low        | Any   | Any        | ❌ WORKFLOW
```

#### Step 2: Use the Decision Tree

[DECISION_TREE.md](DECISION_TREE.md) walks you through **10 structured decision steps**:

```
Question 1: What problem are you solving? → Write one sentence
Question 2: Why not a service? → Check 6 criteria
Question 3: What autonomy level? → Levels 1-5
Question 4: Does it block flow? → Observer/Controller/Hybrid
Question 5: What intelligence? → Levels 0-4
...and 5 more questions
```

**Output**: A complete design document with all architectural decisions made.

#### Step 3: Validate Your Decision

Use the checklist at [DECISION_TREE.md#validation](DECISION_TREE.md#validation):

```
Before building:
  □ Can workflow solve this?
  □ Is value > cost?
  □ Can mistakes be caught?
  □ Is feedback measurable?
```

---

## 🏗️ How to Create an Agent (Step-by-Step)

Once you've identified you need an agent, here's how to build one:

### Phase 1: Design Your Agent (Use the Framework)

#### 1. Complete the Decision Tree

Work through [DECISION_TREE.md](DECISION_TREE.md) to answer:
- What autonomy level? (2, 3, or 4)
- Observer, Controller, or Hybrid?
- What intelligence level? (0-4)
- Architecture: SDK-only, Service-only, or Hybrid?

**Example Output**:
```markdown
Agent: Costing Agent
Classification: Level 2 Reactive, Observer, Intelligence L1
Architecture: Hybrid (SDK + Service), Push model, Async/Buffered
```

#### 2. Design the Core Components

Using [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md):

**Define your Model** (How it thinks):
```typescript
// Start simple - rule-based
if (tokens > threshold) {
  alert("High cost detected");
}
```

**Define your Tools** (What it can do):
```typescript
const tools = {
  trackCost: (event) => { /* ... */ },
  sendAlert: (message) => { /* ... */ },
  generateReport: () => { /* ... */ }
};
```

**Define your Environment** (What it observes):
```typescript
const environment = {
  getCurrentCost: () => db.query('SELECT SUM(cost) ...'),
  getThreshold: () => config.costThreshold,
  getRecentEvents: () => db.query('SELECT * FROM events ...')
};
```

#### 3. Design the Agent Loop

Using the **Core Agent Loop Pattern** from [PATTERNS.md](PATTERNS.md#core-agent-loop-pattern):

```typescript
class MyAgent {
  async pursue(goal: Goal): Promise<Result> {
    let iteration = 0;

    while (iteration < goal.maxIterations) {
      iteration++;

      // PLAN: What to do next?
      const plan = await this.plan(goal, currentState);

      // ACT: Execute
      const result = await this.act(plan);

      // OBSERVE: What happened?
      const observation = await this.observe(result);

      // REFLECT: Did it work?
      if (goal.successCriteria(observation)) {
        return { success: true, iterations: iteration };
      }

      currentState = observation;
    }

    return { success: false, reason: 'Max iterations' };
  }
}
```

#### 4. Select Patterns

From [PATTERNS.md](PATTERNS.md), choose patterns based on your classification:

**For Observer Agents** (like costing-agent):
- ✅ Pattern 1: Auto-Registration
- ✅ Pattern 2: Graceful Shutdown
- ✅ Pattern 3: Heartbeat
- ✅ Pattern 4: Event Buffer
- ✅ Pattern 5: Resilient Transmitter
- ✅ Pattern 12: Hierarchical Tenancy

**For Controller Agents** (like auth-agent):
- ✅ Pattern 1: Auto-Registration
- ✅ Pattern 2: Graceful Shutdown
- ✅ Pattern 10: Middleware
- ✅ Pattern 13: Local Token Validation
- ✅ Pattern 14: Behavioral Profile (if Level 3+)
- ✅ Pattern 15: Risk Scoring (if Level 4)

### Phase 2: Implement (Start Simple)

#### Week 1-2: Build Level 2 (Reactive)

**Goal**: Ship a working agent FAST with simple rules.

1. **Choose a template**:
   - Observer → [templates/passive-observer/](templates/passive-observer/)
   - Controller → [templates/active-controller/](templates/active-controller/)

2. **Implement core SDK**:
```typescript
// SDK entry point
class MyAgent {
  constructor(config: Config) {
    this.config = config;
    this.buffer = new EventBuffer();
    this.transmitter = new ResilientTransitter();
  }

  async init() {
    // Auto-registration (Pattern 1)
    await this.register();
    this.startHeartbeat();
  }

  track(event: Event) {
    // Non-blocking
    this.buffer.add(event);
  }

  async shutdown() {
    // Graceful shutdown (Pattern 2)
    await this.buffer.flush();
    this.stopHeartbeat();
  }
}
```

3. **Implement service**:
```typescript
// Registration endpoint
POST /api/v1/register
{
  projectId: string,
  fingerprint: string,
  metadata: object
}
→ Returns: { deploymentId, config }

// Ingestion endpoint (observers)
POST /api/v1/track
{
  events: Event[]
}
→ Returns: { success: true }
```

4. **Deploy and test**:
   - Deploy service (PostgreSQL + Redis + Node.js)
   - Integrate SDK into ONE test project
   - Validate it works

**Target**: Ship in 2 weeks with simple rule-based logic.

#### Week 3-4: Add Level 3 (Proactive)

**Goal**: Add pattern detection, anomaly alerting.

From [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md#level-2-pattern-detection):

```typescript
async detectAnomalies(metric: Metric): Promise<Anomaly[]> {
  // Get baseline (7-day average)
  const baseline = await this.getBaseline(metric.name, { days: 7 });

  // Threshold check
  if (metric.value > baseline.p95 * 2) {
    return [{
      type: 'threshold_exceeded',
      severity: 'high',
      value: metric.value,
      threshold: baseline.p95 * 2
    }];
  }

  return [];
}
```

**A/B Test**: Run Level 2 and Level 3 side-by-side, compare false positive rates.

#### Week 5-8: Add Level 4 (Autonomous)

**Goal**: Risk scoring, autonomous actions, learning.

From [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md#level-4-autonomous-decision-making):

```typescript
async makeDecision(context: Context): Promise<Decision> {
  // Risk scoring
  const riskScore = await this.calculateRisk(context);

  // Autonomous decision
  if (riskScore > 85) {
    return { action: 'allow', reasoning: 'High trust' };
  } else if (riskScore > 60) {
    return { action: 'challenge_mfa', reasoning: 'Medium trust' };
  } else {
    return { action: 'block', reasoning: 'Low trust' };
  }
}

// Execute autonomously
await this.executeDecision(decision, context);
```

**Gradual Rollout**: Start with 1% traffic, monitor false positives, increase to 100%.

### Phase 3: Optimize & Monitor

#### Add Evaluation Harnesses

From [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#evaluation-harnesses):

```typescript
const scenarios: EvaluationScenario[] = [
  {
    name: "brute-force-attack",
    initialState: { failedLogins: 10, isBlocked: false },
    goal: { successCriteria: (obs) => obs.isBlocked === true },
    expectedOutcome: {
      success: true,
      maxIterations: 2,
      mustUseTools: ["blockUser"]
    }
  },
  // Add 10+ scenarios
];

// Run in CI/CD
const results = await harness.runEvaluation(agent, scenarios);
const passRate = results.filter(r => r.passed).length / results.length;

if (passRate < 0.95) {
  throw new Error("Agent evaluation failed");
}
```

**Target**: 95% pass rate, < 5 avg iterations to goal.

#### Track Agent Metrics

From [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md#operational-metrics):

```typescript
interface AgentMetrics {
  // Loop efficiency
  avgIterationsToGoal: number;      // < 5
  convergenceRate: number;           // > 95%

  // Accuracy
  truePositiveRate: number;         // > 99%
  falsePositiveRate: number;        // < 1%

  // Cost
  llmCostPerGoal: number;           // Track & optimize
  toolInvocationsPerGoal: number;
}
```

**Dashboard**: Visualize these metrics, alert on degradation.

---

## 🎓 Learning Path

### 1. Start Here (30 min)
- Read [README.md](README.md) - Overview & when to use agents
- Read [FRAMEWORK_SUMMARY.md](FRAMEWORK_SUMMARY.md) - One-page visual summary

### 2. Design Fundamentals (2 hours)
- Read [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md) - Model+Tools+Env, Agent Loop, Empathy
- Read [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md) - 6-phase design methodology

### 3. Make Decisions (1 hour)
- Work through [DECISION_TREE.md](DECISION_TREE.md) - Step-by-step decisions for YOUR agent

### 4. Implementation (3 hours)
- Read [PATTERNS.md](PATTERNS.md) - 15 reusable patterns with code
- Copy patterns for your agent type
- Read template README for your classification

### 5. Advanced Topics (2 hours)
- [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) - Level 0-4 intelligence guide
- [MULTI_AGENT_PATTERNS.md](MULTI_AGENT_PATTERNS.md) - When to add second agent (probably never)

**Total**: ~8 hours to go from "What's an agent?" to "I can design and build one."

---

## 📋 Real-World Example: Building a Costing Agent

Let's walk through a complete example:

### Step 1: Identify

**Problem**: "No visibility into LLM API costs across multiple projects"

**Three-Factor Test**:
- Complexity: Exploratory? → NO (data collection is straightforward)
- Value: Worth it? → YES (saves $10k/month in unexpected costs, break-even < 1 week)
- Error Cost: Tolerable? → YES (tracking error = missing cost data, not dangerous)

**Wait, should this be an agent?** Let's check:
- Autonomous behavior? → YES (auto-instruments, self-registers)
- Learning/adaptation? → NO (just aggregation)
- Proactive actions? → NO (just reports)
- Cross-cutting? → YES (all projects)

**Decision**: YES, build a **Level 2 Observer Agent** (Reactive, no intelligence needed).

### Step 2: Design

Work through [DECISION_TREE.md](DECISION_TREE.md):

```markdown
Classification:
- Autonomy: Level 2 (Reactive)
- Mode: Observer (non-blocking)
- Intelligence: L1 (aggregation only)

Architecture:
- Deployment: Hybrid (SDK + Service)
- Communication: Push (events)
- Data Flow: Async, Buffered

Components:
SDK: EventBuffer, Transmitter, Auto-instrumentation
Service: Ingestion, PostgreSQL, Aggregation
```

### Step 3: Implement

**Week 1**:
- SDK: Auto-instrument OpenAI SDK (Pattern 9: Monkey Patching)
- Service: Registration + ingestion endpoints
- Database: PostgreSQL schema (Org → Project → Deployment → Events)

**Week 2**:
- Add: Event Buffer (Pattern 4), Resilient Transmitter (Pattern 5)
- Add: Graceful shutdown (Pattern 2)
- Deploy & test with 1 project

**Result**: Working cost tracking in 2 weeks!

**Week 3-4**: Add anomaly detection (Level 3)
- Alert when daily cost > 2x baseline

**Week 5+**: Optimize, rollout to all projects

### Step 4: Evaluate

```typescript
const scenarios = [
  {
    name: "track-openai-call",
    test: async () => {
      await openai.chat.completions.create({ model: "gpt-4", ... });
      // Wait for buffer flush
      await sleep(15000);
      // Verify event captured
      const events = await db.query("SELECT * FROM events ...");
      assert(events.length > 0);
    }
  }
];
```

**Result**: 100% test pass rate, < 1ms tracking overhead, deployed to 50+ projects.

---

## 🚀 Quick Decision Flowchart

```
START → Is the problem exploratory?
           │
           ├─ NO  → Use Workflow
           │
           └─ YES → Calculate ROI
                       │
                       ├─ ROI < 0 → Don't automate
                       │
                       └─ ROI > 0 → Check error cost
                                      │
                                      ├─ Critical → Human decides
                                      │
                                      └─ Tolerable → BUILD AGENT!
                                                        │
                                                        ↓
                                         Use DECISION_TREE.md
                                                        │
                                                        ↓
                                         Pick template from templates/
                                                        │
                                                        ↓
                                         Copy patterns from PATTERNS.md
                                                        │
                                                        ↓
                                         Start with Level 2 (simple)
                                                        │
                                                        ↓
                                         Ship in 2 weeks!
```

---

## 💡 Pro Tips

1. **Always start with Level 2** - Don't build Level 4 on day one
2. **Use templates** - Don't start from scratch
3. **Copy patterns** - All code is in PATTERNS.md
4. **Measure before optimizing** - Get metrics first
5. **Evaluation harness = must have** - Test agent behavior, not just code
6. **Agent empathy** - Debug from agent's limited perspective
7. **Fail-closed for security** - Deny on error for auth/compliance agents
8. **< 5 minute integration** - If SDK is hard to use, fix the SDK

---

## 🆘 Common Mistakes to Avoid

❌ **Building agents for everything** → Use the three-factor test
❌ **Starting with Level 4** → Start Level 2, evolve
❌ **No evaluation harness** → Can't measure if agent works
❌ **Ignoring agent empathy** → Debugging becomes impossible
❌ **No loop metrics** → Can't optimize without data
❌ **Adding second agent too early** → Most problems don't need multiple agents
❌ **Fail-open for security** → Security agents must fail-closed
❌ **Vague success criteria** → Agent can't verify if goal achieved

---

## 📚 Next Steps

1. **Read** [README.md](README.md) and [FRAMEWORK_SUMMARY.md](FRAMEWORK_SUMMARY.md)
2. **Identify** your agent using the three-factor test
3. **Design** using [DECISION_TREE.md](DECISION_TREE.md)
4. **Implement** using patterns from [PATTERNS.md](PATTERNS.md)
5. **Evaluate** using harnesses from [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md)
6. **Ship** and iterate!

---

**Remember**: Most problems don't need agents. But when they do, this framework helps you build them right the first time.
