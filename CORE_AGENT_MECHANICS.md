# Core Agent Mechanics

The foundational principles and patterns that govern how agents think, operate, and improve.

## Overview

While the framework's other documents cover **infrastructure** (deployment, resilience, multi-tenancy) and **intelligence** (risk scoring, behavioral analysis), this document focuses on **how agents operate at their core**:

0. **Agent Composition**: The fundamental building blocks (Model + Tools + Environment)
1. **The Agent Loop**: How agents continuously pursue goals
2. **Tool Design**: How to equip agents with capabilities
3. **Verification**: How agents know they succeeded
4. **Incremental Development**: How to start simple and evolve
5. **Operational Metrics**: How to measure agent effectiveness
6. **Agent Empathy**: Understanding the agent's perspective
7. **Evaluation Harnesses**: Testing agent behavior
8. **Prompt Engineering**: Iterating on goals and behavior

---

## 0. Agent Composition: Model + Tools + Environment

**Every agent, at its core, is composed of three elements:**

```
┌──────────────────────────────────────────┐
│              THE AGENT                    │
├──────────────────────────────────────────┤
│                                           │
│  ┌─────────┐  ┌────────┐  ┌────────────┐│
│  │  MODEL  │  │ TOOLS  │  │ENVIRONMENT ││
│  │         │  │        │  │            ││
│  │ Reasons │→ │ Acts   │→ │ Observes   ││
│  │ Decides │  │ Calls  │  │ Provides   ││
│  │ Plans   │  │ Does   │  │ Feedback   ││
│  └─────────┘  └────────┘  └────────────┘│
│       ↑            ↑            │        │
│       │            │            │        │
│       └────────────┴────────────┘        │
│              (Feedback Loop)              │
└──────────────────────────────────────────┘
```

### The Three Components

#### 1. **Model** - The Brain

The reasoning engine that makes decisions. Can be:
- **Rule-based** (Level 2): Simple if/then logic
  ```typescript
  if (failedLogins > 5) { blockUser(); }
  ```

- **Pattern-based** (Level 3): Statistical analysis
  ```typescript
  if (currentRate > baseline.mean + 3*stdDev) { alert(); }
  ```

- **ML-based** (Level 4): Risk scoring, neural networks
  ```typescript
  const riskScore = await riskModel.predict(features);
  ```

- **LLM-based** (Future): Natural language reasoning
  ```typescript
  const decision = await llm.decide(prompt, context);
  ```

**Key principle:** Start with the simplest model that works. Most agents don't need LLMs!

#### 2. **Tools** - The Hands

Actions the agent can take to affect the world:
- Function calls (blockUser, deployService, sendAlert)
- API requests (database queries, external services)
- System commands (file operations, shell commands)
- LLM calls (for LLM-enhanced agents)

**Example tool set for Auth Agent:**
```typescript
const tools = {
  blockUser: async (userId: string, duration: number) => {...},
  sendMFAChallenge: async (userId: string) => {...},
  logSecurityEvent: async (event: SecurityEvent) => {...},
  queryFailedLogins: async (userId: string) => {...},
  blockIP: async (ipAddress: string) => {...}
};
```

**Key principle:** Tools should be idempotent, fast, and have clear success/failure signals.

#### 3. **Environment** - The World

Everything the agent can observe:
- **State**: Current system state (user data, metrics, logs)
- **Feedback**: Results from tool invocations
- **Context**: Historical data, trends, patterns
- **External signals**: Threat intelligence, third-party APIs

**Example environment for Auth Agent:**
```typescript
const environment = {
  // Current state
  getUser: async (userId: string) => User,
  getFailedLogins: async (userId: string) => number,

  // Historical context
  getUserBehavior: async (userId: string) => BehaviorProfile,
  getThreatIntel: async () => ThreatData,

  // Feedback from actions
  wasBlockEffective: async (blockId: string) => boolean,
  getBlockedAttempts: async (userId: string) => number
};
```

**Key principle:** The environment defines what the agent can "see". Limited visibility = limited decision-making.

### Why This Mental Model Matters

**When building an agent, ask:**

1. **Model**: How should it think?
   - Rule-based is often sufficient
   - Don't default to LLMs (expensive, slower, less predictable)
   - Match model complexity to problem complexity

2. **Tools**: What can it do?
   - Start with minimal tool set
   - Each tool increases complexity
   - Tools should be composable

3. **Environment**: What can it see?
   - Limited context = limited decisions
   - Too much context = overwhelming (especially for LLMs)
   - Provide just enough information

### Iterating on Agent Design

**The beauty of this model:** You can iterate on each component independently.

```typescript
// Iteration 1: Simple rule-based agent
const agentV1 = {
  model: ruleBasedDecisionEngine,
  tools: [blockUser, logEvent],
  environment: { getFailedLogins }
};

// Iteration 2: Add more tools
const agentV2 = {
  model: ruleBasedDecisionEngine,  // Same model
  tools: [blockUser, logEvent, sendMFAChallenge, blockIP],  // More tools
  environment: { getFailedLogins }  // Same environment
};

// Iteration 3: Smarter model
const agentV3 = {
  model: riskScoringEngine,  // Upgraded model
  tools: [blockUser, logEvent, sendMFAChallenge, blockIP],  // Same tools
  environment: {  // Richer environment
    getFailedLogins,
    getUserBehavior,
    getThreatIntel
  }
};
```

**Start simple:**
- Model: Rule-based
- Tools: 2-3 essential actions
- Environment: Minimal state

**Evolve intentionally:**
- Add tools when needed
- Upgrade model when rules become complex
- Enrich environment when more context helps

### Example: Auth Agent Evolution

**Week 1 (MVP):**
```typescript
{
  model: "if failedLogins > 5 → block",
  tools: [blockUser],
  environment: { getFailedLogins }
}
```

**Week 3 (Pattern detection):**
```typescript
{
  model: "if rate > baseline + 3σ → block",
  tools: [blockUser, sendAlert],
  environment: { getFailedLogins, getUserBehavior }
}
```

**Week 6 (Risk-based):**
```typescript
{
  model: "riskScore = f(rate, location, device, threats)",
  tools: [blockUser, sendMFAChallenge, blockIP, sendAlert],
  environment: {
    getFailedLogins,
    getUserBehavior,
    getThreatIntel,
    getDeviceInfo,
    getGeoLocation
  }
}
```

**Complexity later. Clarity first.**

---

## 1. The Core Agent Loop

Every agent, regardless of complexity, operates on this fundamental cycle:

```
┌─────────────────────────────────────────────┐
│                 AGENT LOOP                   │
└─────────────────────────────────────────────┘

    ┌──────┐
    │ GOAL │ ──► Define success criteria
    └───┬──┘     (tests pass, metric threshold)
        │
        ▼
    ┌──────┐
    │ PLAN │ ──► Break goal into steps
    └───┬──┘     (what actions to take)
        │
        ▼
    ┌──────┐
    │ ACT  │ ──► Execute tools/actions
    └───┬──┘     (call APIs, modify state)
        │
        ▼
    ┌─────────┐
    │ OBSERVE │ ──► Collect feedback
    └────┬────┘     (results, metrics, errors)
         │
         ▼
    ┌─────────┐
    │ REFLECT │ ──► Analyze outcome
    └────┬────┘     (did it work? what changed?)
         │
         ▼
    ┌────────────────┐
    │ REPEAT or EXIT │ ──► Continue or stop
    └────────────────┘     (goal achieved or max iterations)
         │
         └──────────────► (loop back to PLAN)
```

### Why This Matters

**Without an explicit loop:**
- Agents don't know when to stop
- No clear success criteria
- Can't measure iterations or convergence
- Hard to debug failures

**With the loop:**
- Clear goal and exit conditions
- Measurable progress (iterations, convergence rate)
- Debuggable (which step failed?)
- Improvable (optimize loop efficiency)

---

## 2. The Loop in Practice

### Example: Auth Agent - Brute Force Detection

```typescript
class BruteForceDetector {
  async detectAndRespond(userId: string): Promise<void> {
    // GOAL: Prevent brute force attack
    const goal = {
      description: "Detect and stop brute force attack",
      successCriteria: "failed_logins < threshold AND attacker_blocked",
      maxIterations: 3
    };

    let iteration = 0;
    let goalAchieved = false;

    while (!goalAchieved && iteration < goal.maxIterations) {
      iteration++;

      // PLAN: Determine next action
      const plan = await this.planNextAction(userId);

      // ACT: Execute action
      const action = await this.executeAction(plan);

      // OBSERVE: Collect results
      const observation = await this.observeOutcome(userId);

      // REFLECT: Analyze if goal achieved
      const reflection = await this.reflect(observation, goal);

      goalAchieved = reflection.goalAchieved;

      if (!goalAchieved && reflection.shouldAdjustStrategy) {
        // Adjust for next iteration
        await this.adjustStrategy(reflection.learnings);
      }
    }

    // Log loop metrics
    this.metrics.recordLoopCompletion({
      iterations: iteration,
      goalAchieved,
      convergenceRate: goalAchieved ? 1 : 0
    });
  }

  async planNextAction(userId: string): Promise<Plan> {
    const context = await this.getContext(userId);

    // Decide what to do based on current state
    if (context.failedLogins >= 5 && !context.isBlocked) {
      return { action: 'BLOCK_USER', reason: 'Exceeded threshold' };
    } else if (context.isBlocked && context.timeSinceBlock > 3600) {
      return { action: 'UNBLOCK_USER', reason: 'Cooldown period passed' };
    } else {
      return { action: 'MONITOR', reason: 'Within normal range' };
    }
  }

  async reflect(observation: Observation, goal: Goal): Promise<Reflection> {
    // Did we achieve the goal?
    const goalAchieved =
      observation.failedLogins < 5 &&
      observation.attackerBlocked;

    // What did we learn?
    const learnings = {
      actionEffective: observation.failedLoginsDecreased,
      sideEffects: observation.legitimateUsersAffected,
      shouldEscalate: observation.attackPersists
    };

    return {
      goalAchieved,
      shouldAdjustStrategy: !goalAchieved && learnings.actionEffective === false,
      learnings
    };
  }
}
```

### Example: Deployment Agent - Gradual Rollout

```typescript
class DeploymentAgent {
  async deploy(version: string): Promise<DeploymentResult> {
    // GOAL: Deploy new version with < 0.1% error rate
    const goal = {
      description: "Deploy v2.0 safely",
      successCriteria: "deployed=100% AND error_rate < 0.1%",
      maxIterations: 10
    };

    let currentRollout = 0; // Start at 0%
    let iteration = 0;

    while (currentRollout < 100 && iteration < goal.maxIterations) {
      iteration++;

      // PLAN: Determine next rollout percentage
      const nextRollout = this.planNextIncrement(currentRollout);

      // ACT: Deploy to next percentage
      await this.deployToPercentage(version, nextRollout);

      // OBSERVE: Wait and collect metrics
      await this.wait(120000); // 2 minutes
      const metrics = await this.observeMetrics(version);

      // REFLECT: Is it safe to continue?
      const reflection = await this.reflect(metrics, goal);

      if (reflection.shouldRollback) {
        await this.rollback(version);
        return { success: false, reason: reflection.reason };
      }

      if (reflection.canProceed) {
        currentRollout = nextRollout;
      } else {
        // Pause and wait before retrying
        await this.wait(300000); // 5 minutes
      }
    }

    return {
      success: currentRollout === 100,
      iterations: iteration
    };
  }
}
```

---

## 3. Tool Design Principles

Agents need **tools** (functions, APIs, actions) to accomplish goals. Well-designed tools make agents more effective.

### Tool Specification Format

Every tool should have:

```typescript
interface AgentTool {
  // Identity
  name: string;
  description: string;

  // Interface
  parameters: Record<string, ParameterSpec>;
  returnType: TypeSpec;

  // Constraints
  rateLimit?: { requests: number; window: string };
  permissions?: string[];
  sideEffects: 'none' | 'read-only' | 'mutating' | 'dangerous';

  // Safety
  idempotent: boolean;
  errorCost: 'low' | 'medium' | 'high' | 'critical';
  requiresApproval: boolean;

  // Verification
  successCriteria: string;
  failureConditions: string[];

  // Documentation
  examples: Example[];
}
```

### Example: Block User Tool

```typescript
const blockUserTool: AgentTool = {
  name: "blockUser",
  description: "Temporarily block a user from authentication",

  parameters: {
    userId: { type: "string", required: true },
    reason: { type: "string", required: true },
    durationMinutes: { type: "number", default: 60, max: 1440 }
  },

  returnType: { success: "boolean", expiresAt: "timestamp" },

  rateLimit: { requests: 100, window: "1m" },
  permissions: ["auth:block"],
  sideEffects: "dangerous", // Affects user access

  idempotent: true, // Blocking twice = same result
  errorCost: "high", // False positive blocks legitimate users
  requiresApproval: false, // Autonomous for high-confidence threats

  successCriteria: "User cannot authenticate until expiry",
  failureConditions: [
    "User is admin (cannot block admins)",
    "User already blocked (no-op)"
  ],

  examples: [
    {
      input: { userId: "user-123", reason: "Brute force detected", durationMinutes: 60 },
      output: { success: true, expiresAt: "2025-01-15T10:00:00Z" }
    }
  ]
};
```

### Tool Design Best Practices

**1. Clear Interfaces**
- Explicit parameters and return types
- No ambiguous behavior
- Agent understands what tool does from description

**2. Safety Constraints**
- Rate limits to prevent abuse
- Permission checks
- Error cost classification
- Approval requirements for high-stakes actions

**3. Idempotency**
- Safe to retry on failure
- Calling twice produces same result
- Critical for autonomous agents

**4. Fast Feedback**
- Tool returns quickly (< 5s preferred)
- Clear success/failure signals
- Actionable error messages

**5. Composability**
- Tools can be chained together
- Outputs become inputs to other tools
- Minimal side effects unless necessary

---

## 4. Verification Mechanisms

How does an agent know it succeeded? Verification is **critical** for autonomous operation.

### Types of Verification

#### A. Automated Verification (Preferred)

**Unit Tests:**
```typescript
// Agent's goal: "Make the user registration test pass"
await runTest('user-registration.test.ts');
// Success criteria: exit code 0, all assertions pass
```

**Integration Tests:**
```typescript
// Agent's goal: "Deploy service with < 1% error rate"
const metrics = await getMetrics();
const verified = metrics.errorRate < 0.01;
```

**Property Checks:**
```typescript
// Agent's goal: "Optimize cache hit rate"
const before = await getCacheHitRate(); // 75%
await agent.optimizeCache();
const after = await getCacheHitRate(); // 85%
const verified = after > before; // Improved
```

**External Oracles:**
```typescript
// Agent's goal: "Block all known malicious IPs"
const maliciousIPs = await getThreatIntelligence();
const blockedIPs = await getBlockedIPs();
const verified = maliciousIPs.every(ip => blockedIPs.includes(ip));
```

#### B. Human Verification (When Necessary)

**Human-in-the-Loop:**
```typescript
class ApprovalRequiredTool {
  async execute(params: any): Promise<Result> {
    // Agent proposes action
    const proposal = {
      action: "delete_user_data",
      params,
      reasoning: "User requested data deletion (GDPR)",
      impact: "HIGH - Irreversible"
    };

    // Request human approval
    const approved = await this.requestApproval(proposal);

    if (!approved) {
      return { success: false, reason: "Approval denied" };
    }

    // Execute with approval
    return await this.executeAction(params);
  }
}
```

**Confidence Thresholds:**
```typescript
// Agent decides based on confidence
const decision = await agent.makeDecision(context);

if (decision.confidence > 0.95) {
  // High confidence: Execute autonomously
  await decision.execute();
} else if (decision.confidence > 0.7) {
  // Medium confidence: Log and ask for review
  await this.requestReview(decision);
} else {
  // Low confidence: Skip action
  this.log.warn("Decision confidence too low", decision);
}
```

### Verification in the Loop

```typescript
async function agentLoop(goal: Goal): Promise<LoopResult> {
  while (!goalAchieved) {
    const action = await plan();
    const result = await execute(action);

    // VERIFY: Did the action work?
    const verification = await verify(result, goal);

    if (verification.passed) {
      goalAchieved = true;
    } else if (verification.canRetry) {
      // Try different approach
      await adjustStrategy(verification.feedback);
    } else {
      // Unrecoverable failure
      return { success: false, reason: verification.reason };
    }
  }

  return { success: true, iterations };
}
```

---

## 5. Incremental Development Path

**Start simple, evolve complexity as needed.**

### Phase 1: Rule-Based (Level 2 - Reactive)

**Characteristics:**
- Deterministic if/then logic
- No machine learning
- Fixed thresholds
- Static policies

**Example: Rate Limiting**
```typescript
if (requestCount > 100) {
  blockUser(userId, "Rate limit exceeded");
}
```

**When to use:**
- Problem is well-understood
- Rules are clear and stable
- Fast and predictable required

### Phase 2: Pattern Detection (Level 3 - Proactive)

**Characteristics:**
- Learns normal patterns
- Detects anomalies
- Dynamic thresholds
- Behavioral baselines

**Example: Adaptive Rate Limiting**
```typescript
const userBaseline = await learnUserPattern(userId); // Avg: 50 req/hour
const currentRate = await getCurrentRate(userId); // Now: 200 req/hour

if (currentRate > userBaseline.mean + (3 * userBaseline.stdDev)) {
  // Anomaly: 4x normal rate
  blockUser(userId, "Unusual activity detected");
}
```

**When to use:**
- Patterns change over time
- Static thresholds cause false positives
- Need personalization

### Phase 3: Autonomous Decision-Making (Level 4 - Autonomous)

**Characteristics:**
- Risk scoring
- Multi-factor decisions
- Autonomous actions
- Continuous learning

**Example: Intelligent Threat Response**
```typescript
const riskScore = await calculateRisk({
  requestRate: currentRate,
  userHistory: userBaseline,
  deviceFingerprint: device,
  geolocation: location,
  threatIntel: globalThreats
});

if (riskScore > 80) {
  await blockUser(userId);
} else if (riskScore > 60) {
  await requireMFA(userId);
} else if (riskScore > 40) {
  await logAlert(userId);
}

// Learn from outcome
await updateModel(riskScore, actualThreat);
```

**When to use:**
- Multiple factors influence decision
- Need autonomous response
- Want continuous improvement

### Phase 4: LLM-Enhanced (Future)

**Characteristics:**
- Natural language reasoning
- Complex context understanding
- Few-shot learning
- Explainable decisions

**Example: Context-Aware Security**
```typescript
const decision = await llm.decide({
  prompt: `
    User login attempt:
    - Email: ${email}
    - Location: ${location} (unusual: ${isUnusual})
    - Device: ${device} (recognized: ${isKnown})
    - Recent activity: ${recentActivity}
    - Threat intel: ${threatLevel}

    Should we: (1) Allow, (2) Challenge with MFA, or (3) Block?
    Explain your reasoning.
  `,
  temperature: 0.1, // Deterministic for security
});

// Execute with explanation
await executeDecision(decision.action, decision.reasoning);
```

**When to use:**
- Complex context interpretation needed
- Rules too numerous to maintain
- Want natural explainability
- Models are reliable enough

### Evolution Strategy

**Don't build Level 4 first!**

```
Week 1-2:  Build Level 2 (rule-based) ──► Ship to production
Week 3-4:  Add Level 3 (pattern detection) ──► A/B test
Week 5-8:  Add Level 4 (autonomous decisions) ──► Gradual rollout
Month 3+:  Optimize, add LLM if needed
```

**Benefits:**
- Fast initial value delivery
- Learn from production data
- Reduce complexity risk
- Easier debugging

---

## 6. Operational Metrics

How to measure agent effectiveness beyond traditional performance metrics.

### Traditional Performance Metrics

These apply to **all agents**:

```typescript
interface PerformanceMetrics {
  // Latency
  p50_latency_ms: number;  // Median response time
  p95_latency_ms: number;  // 95th percentile (target: < 50ms for validation)
  p99_latency_ms: number;  // 99th percentile

  // Throughput
  requests_per_second: number;  // Target: 10,000+

  // Availability
  uptime_percentage: number;    // Target: 99.9%
  error_rate_percentage: number; // Target: < 0.1%
}
```

### Agent-Specific Metrics

These measure **agent behavior** and effectiveness:

```typescript
interface AgentMetrics {
  // Loop Efficiency
  avg_iterations_to_goal: number;        // How many loops to succeed (target: < 5)
  convergence_rate: number;              // % of goals achieved (target: > 95%)
  max_iterations_reached_count: number;  // How often agent hits iteration limit

  // Accuracy
  true_positive_rate: number;   // % of real threats caught (target: > 99%)
  false_positive_rate: number;  // % of false alarms (target: < 1%)
  precision: number;            // TP / (TP + FP)
  recall: number;               // TP / (TP + FN)

  // Cost
  llm_api_cost_per_goal: number;        // $ per successful goal
  tool_invocations_per_goal: number;    // Tool calls per goal
  total_cost_per_day: number;           // Operational cost

  // Improvement Over Time
  accuracy_trend_weekly: number[];      // Is agent getting better?
  false_positive_trend_weekly: number[]; // Are mistakes decreasing?

  // Learning
  model_updates_per_week: number;       // How often does agent learn?
  policy_adjustments_per_week: number;  // How often do rules change?
}
```

### Example: Tracking Loop Metrics

```typescript
class AgentMetricsTracker {
  async trackLoopExecution(
    goalId: string,
    iterations: number,
    success: boolean,
    cost: number
  ): Promise<void> {
    await this.db.insert('agent_loop_metrics', {
      goal_id: goalId,
      iterations,
      success,
      cost,
      timestamp: new Date()
    });

    // Update rolling averages
    await this.updateAggregates();
  }

  async getAgentHealth(): Promise<AgentHealth> {
    const last24h = await this.getMetrics('24h');

    return {
      convergenceRate: last24h.successCount / last24h.totalGoals,
      avgIterations: last24h.totalIterations / last24h.successCount,
      costPerGoal: last24h.totalCost / last24h.successCount,

      // Health indicators
      healthy: last24h.convergenceRate > 0.95 && last24h.avgIterations < 5,
      alerts: this.generateAlerts(last24h)
    };
  }

  generateAlerts(metrics: Metrics): Alert[] {
    const alerts = [];

    if (metrics.convergenceRate < 0.9) {
      alerts.push({
        severity: 'high',
        message: 'Convergence rate dropped below 90%',
        action: 'Review goal definitions and tool effectiveness'
      });
    }

    if (metrics.avgIterations > 10) {
      alerts.push({
        severity: 'medium',
        message: 'Average iterations increased significantly',
        action: 'Agent may be inefficient, review loop logic'
      });
    }

    return alerts;
  }
}
```

### Metrics Dashboard

Track these visually:

```
Agent Health Dashboard
======================

Loop Efficiency               Accuracy
  Avg Iterations: 3.2 ✅       True Positive:  99.2% ✅
  Convergence:    96.1% ✅       False Positive: 0.8% ✅
  Max Iterations: 12 events ⚠️  Precision:      99.2% ✅

Cost                          Improvement (7 days)
  Per Goal:  $0.03 ✅           Accuracy:    +0.3% ↗️
  Per Day:   $124  ✅           Iterations:  -0.5   ↗️
  LLM Calls: 2.1/goal ✅        Cost:        -$0.01 ↗️

Recent Alerts
  ⚠️  Max iterations reached 12 times in last hour
  ℹ️  Model updated with 1,247 new examples
```

---

## 7. Agent Empathy: Think Like Your Agent

**Critical Principle: Debug from the agent's perspective, not yours.**

When agents fail, developers often debug from their own perspective ("Why didn't it do what I wanted?"). Instead, debug from the agent's perspective ("What information did it have? What made sense given its limited view?").

### The Context Window Problem

**Your view vs. Agent's view:**

```
┌─────────────────────────────────────────┐
│         DEVELOPER VIEW (Full)            │
├─────────────────────────────────────────┤
│ • Complete codebase                      │
│ • Git history                            │
│ • Documentation                          │
│ • Team conversations                     │
│ • System architecture                    │
│ • Business context                       │
│ • Future plans                           │
└─────────────────────────────────────────┘
                   vs.
┌─────────────────────────────────────────┐
│         AGENT VIEW (Limited)             │
├─────────────────────────────────────────┤
│ • Only what's in context window          │
│ • Only tools it has access to            │
│ • Only environment state it can query    │
│ • No memory beyond context               │
│ • No intuition or common sense           │
└─────────────────────────────────────────┘
```

**For LLM-based agents**, the context window is literal (e.g., 128K tokens). For rule-based agents, it's the state and tools available.

### Debugging Questions

When an agent fails, ask from its perspective:

#### 1. **What information did the agent have?**

❌ **Bad debugging**: "Why didn't it know the deployment failed?"
✅ **Good debugging**: "Did the agent have access to deployment status in its environment?"

```typescript
// Agent's environment
const environment = {
  getFailedLogins: async () => number,
  // Missing: getDeploymentStatus ← This is why it didn't know!
};

// Fix: Add deployment status to environment
const fixedEnvironment = {
  getFailedLogins: async () => number,
  getDeploymentStatus: async () => DeploymentStatus, // Now visible
};
```

#### 2. **What tools did the agent have?**

❌ **Bad debugging**: "Why didn't it roll back the deployment?"
✅ **Good debugging**: "Did the agent have a rollback tool?"

```typescript
// Agent's tools
const tools = {
  deploy: async (version) => {...},
  // Missing: rollback tool ← This is why it couldn't rollback!
};

// Fix: Add rollback tool
const fixedTools = {
  deploy: async (version) => {...},
  rollback: async (version) => {...}, // Now available
};
```

#### 3. **Was the goal clear from the agent's perspective?**

❌ **Bad debugging**: "Why didn't it optimize performance?"
✅ **Good debugging**: "Was 'optimize performance' measurably defined?"

```typescript
// Vague goal (agent can't verify success)
const vagueGoal = {
  description: "Optimize performance"  // How? By what measure?
};

// Clear goal (agent can verify)
const clearGoal = {
  description: "Reduce p95 latency below 100ms",
  successCriteria: (obs) => obs.p95Latency < 100,
  maxIterations: 5
};
```

#### 4. **Did feedback make sense to the agent?**

❌ **Bad debugging**: "Why did it keep retrying a failed action?"
✅ **Good debugging**: "Did the tool return a clear failure signal?"

```typescript
// Ambiguous tool response
async function deployService(version: string): Promise<any> {
  // Returns null on error? Or empty object? Or throws?
  // Agent can't tell success from failure!
}

// Clear tool response
async function deployService(version: string): Promise<DeployResult> {
  return {
    success: boolean,
    error?: string,
    deploymentId?: string
  };
  // Agent knows exactly what happened
}
```

### The Empathy Exercise

**When debugging, reconstruct the agent's "world view":**

```typescript
// Exercise: What does the agent know at each step?

// Iteration 1
const agentKnowledge_iter1 = {
  goal: "Stop brute force attack",
  state: { failedLogins: 8 },  // From environment
  tools: ["blockUser", "sendAlert"],
  previousActions: [],
  decision: "blockUser" // Makes sense given failedLogins > 5
};

// Iteration 2 (after blocking)
const agentKnowledge_iter2 = {
  goal: "Stop brute force attack",
  state: { failedLogins: 8 },  // Still 8! Why?
  tools: ["blockUser", "sendAlert"],
  previousActions: [{ action: "blockUser", result: "success" }],
  decision: "blockUser" // Blocks again! Seems stuck!
};

// Problem diagnosis: Agent doesn't see updated failedLogins count
// because environment doesn't refresh state after blocking.

// Fix: Environment should return current state
const fixedEnvironment = {
  getFailedLogins: async (userId: string) => {
    // Returns current count, accounting for recent blocks
    return await db.countFailedLoginsSinceLastBlock(userId);
  }
};
```

### Common Empathy Failures

#### Failure 1: **Assuming Agent Has Context It Doesn't**

```typescript
// Developer thinks:
"The agent should know that user-123 is a test user and not block them."

// Agent actually knows:
{
  userId: "user-123",
  failedLogins: 10  // Just numbers, no context about "test user"
}

// Fix: Add context to environment
environment.isTestUser = async (userId) => boolean;
```

#### Failure 2: **Expecting Intuition or Common Sense**

```typescript
// Developer thinks:
"Obviously don't deploy on Friday afternoon."

// Agent actually knows:
{
  currentTime: "2025-01-10T16:30:00Z"  // Just a timestamp
}

// Fix: Encode business rules explicitly
if (isFridayAfternoon(currentTime)) {
  return { shouldDeploy: false, reason: "Avoid Friday deploys" };
}
```

#### Failure 3: **Vague Success Criteria**

```typescript
// Developer thinks:
"Agent should make the code 'better'."

// Agent actually knows:
"What does 'better' mean? Faster? Shorter? More readable?"

// Fix: Define "better" measurably
successCriteria: (obs) =>
  obs.testsPassed === true &&
  obs.codeLength < originalLength * 0.8 &&
  obs.cyclomaticComplexity < 10
```

### Debugging Checklist

When an agent behaves unexpectedly:

- [ ] **Visibility**: Did the agent have access to the information it needed?
- [ ] **Tools**: Did the agent have the capabilities to act on that information?
- [ ] **Clarity**: Was the goal measurably defined from the agent's perspective?
- [ ] **Feedback**: Did tools return clear success/failure signals?
- [ ] **Context**: Did the agent have enough context, but not too much?
- [ ] **State Refresh**: Did the environment update after agent actions?

### Empathy in LLM Agents (Future)

For LLM-enhanced agents, additional considerations:

**1. Token Limits**
```typescript
// Bad: Entire codebase in prompt (exceeds context window)
const prompt = `Here's the entire codebase:\n${allFiles}...`;

// Good: Only relevant files
const prompt = `Here are the files related to authentication:\n${authFiles}...`;
```

**2. Prompt Clarity**
```typescript
// Bad: Vague instruction
const prompt = "Fix the bugs";

// Good: Specific goal + success criteria
const prompt = `
Goal: Fix failing tests in auth module
Success criteria: All tests in tests/auth/*.test.ts pass
Available tools: editFile, runTests, readFile
Max iterations: 5
`;
```

**3. Tool Descriptions**
```typescript
// Bad: Agent doesn't understand when to use tool
{
  name: "doThing",
  parameters: {...}
}

// Good: Clear description
{
  name: "deployService",
  description: "Deploys a service version to production. Use when all tests pass and approval is granted. Do NOT use on Fridays or before major holidays.",
  parameters: {...}
}
```

### Summary: Think Like Your Agent

✅ **Do:**
- Debug from agent's limited perspective
- Verify agent has visibility into needed state
- Ensure tools return clear signals
- Define goals measurably
- Provide just enough context

❌ **Don't:**
- Assume agent has your full knowledge
- Expect common sense or intuition
- Use vague success criteria
- Overload context (especially for LLMs)
- Forget to refresh state after actions

**Remember**: The agent only knows what you explicitly give it through Model, Tools, and Environment.

---

## 8. Evaluation Harnesses

**Agents need automated testing, just like code needs unit tests.**

An **evaluation harness** is an automated system for testing agent behavior across multiple scenarios.

### Why Evaluation Harnesses Matter

**Problem**: Agents are non-deterministic and exploratory. Manual testing is:
- Time-consuming
- Not reproducible
- Misses edge cases
- Doesn't scale

**Solution**: Automated evaluation harnesses that run agents through test scenarios and verify outcomes.

### Basic Evaluation Harness Structure

```typescript
interface EvaluationScenario {
  name: string;
  description: string;
  initialState: any;
  goal: Goal;
  expectedOutcome: {
    success: boolean;
    maxIterations?: number;
    mustUseTools?: string[];
    mustNotUseTools?: string[];
    finalStateConditions?: (state: any) => boolean;
  };
}

interface EvaluationResult {
  scenario: string;
  passed: boolean;
  iterations: number;
  toolsUsed: string[];
  finalState: any;
  reason?: string;
}

class AgentEvaluationHarness {
  async runEvaluation(
    agent: Agent,
    scenarios: EvaluationScenario[]
  ): Promise<EvaluationResult[]> {
    const results: EvaluationResult[] = [];

    for (const scenario of scenarios) {
      // Reset agent to initial state
      agent.reset(scenario.initialState);

      // Run agent on scenario
      const result = await agent.pursue(scenario.goal);

      // Verify outcome
      const passed = this.verify(result, scenario.expectedOutcome);

      results.push({
        scenario: scenario.name,
        passed,
        iterations: result.iterations,
        toolsUsed: result.toolsUsed,
        finalState: result.finalState,
        reason: passed ? undefined : this.explainFailure(result, scenario)
      });
    }

    return results;
  }
}
```

### Example: Auth Agent Evaluation Harness

```typescript
const authAgentScenarios: EvaluationScenario[] = [
  // Scenario 1: Block brute force attacker
  {
    name: "brute-force-attack",
    description: "Detect and block user with 10 failed logins",
    initialState: {
      userId: "attacker-123",
      failedLogins: 10,
      isBlocked: false
    },
    goal: {
      description: "Stop brute force attack",
      successCriteria: (obs) => obs.isBlocked === true,
      maxIterations: 3
    },
    expectedOutcome: {
      success: true,
      maxIterations: 2,  // Should complete in ≤ 2 iterations
      mustUseTools: ["blockUser"],  // Must use block tool
      finalStateConditions: (state) => state.isBlocked === true
    }
  },

  // Scenario 2: Don't block legitimate user with slow typos
  {
    name: "legitimate-user-typos",
    description: "Don't block user with 3 failed logins over 30 minutes",
    initialState: {
      userId: "user-456",
      failedLogins: 3,
      timeWindow: 1800,  // 30 minutes
      isBlocked: false
    },
    goal: {
      description: "Assess if blocking needed",
      successCriteria: (obs) => obs.decisionMade === true,
      maxIterations: 2
    },
    expectedOutcome: {
      success: true,
      maxIterations: 2,
      mustNotUseTools: ["blockUser"],  // Should NOT block
      finalStateConditions: (state) => state.isBlocked === false
    }
  },

  // Scenario 3: Escalate to MFA for suspicious activity
  {
    name: "suspicious-mfa-challenge",
    description: "Challenge user with MFA for login from new location",
    initialState: {
      userId: "user-789",
      newLocation: true,
      newDevice: false,
      failedLogins: 0
    },
    goal: {
      description: "Assess authentication risk",
      successCriteria: (obs) => obs.mfaChallenged === true,
      maxIterations: 2
    },
    expectedOutcome: {
      success: true,
      mustUseTools: ["sendMFAChallenge"],
      mustNotUseTools: ["blockUser"],  // Should challenge, not block
    }
  }
];

// Run evaluation
const harness = new AgentEvaluationHarness();
const results = await harness.runEvaluation(authAgent, authAgentScenarios);

// Report
console.log(`Passed: ${results.filter(r => r.passed).length}/${results.length}`);
results.filter(r => !r.passed).forEach(r => {
  console.log(`❌ Failed: ${r.scenario} - ${r.reason}`);
});
```

### Evaluation Metrics

Track these across your test scenarios:

```typescript
interface HarnessMetrics {
  // Pass rate
  totalScenarios: number;
  passedScenarios: number;
  failedScenarios: number;
  passRate: number;  // Target: > 95%

  // Efficiency
  avgIterationsPerScenario: number;  // Target: < 5
  maxIterationsExceeded: number;     // Should be 0

  // Tool usage
  toolUsageCorrect: number;          // Used right tools
  toolUsageIncorrect: number;        // Used wrong tools

  // Reliability
  deterministicBehavior: boolean;    // Same scenario → same result
  regressions: number;               // Previously passing scenarios now failing
}
```

### Integration with CI/CD

```typescript
// In your CI pipeline
async function runAgentTests() {
  const harness = new AgentEvaluationHarness();
  const results = await harness.runEvaluation(agent, scenarios);

  const passRate = results.filter(r => r.passed).length / results.length;

  if (passRate < 0.95) {
    throw new Error(`Agent evaluation failed: ${passRate * 100}% pass rate`);
  }

  console.log(`✅ Agent evaluation passed: ${passRate * 100}% pass rate`);
}
```

### When to Add Evaluation Harnesses

**Add early** - Don't wait until agent is complex!

```
Week 1: Build agent + 3 basic scenarios
Week 2: Add 5 more scenarios as you find edge cases
Week 3: Run harness in CI
Week 4+: Add scenario for every bug found
```

### Benefits

- ✅ **Catch regressions**: New changes don't break existing behavior
- ✅ **Document behavior**: Scenarios serve as living documentation
- ✅ **Enable refactoring**: Refactor with confidence
- ✅ **Measure progress**: Track pass rate over time
- ✅ **Faster debugging**: Failing scenario pinpoints issue

**Remember**: Agents without evaluation harnesses are "hope-driven development." Add them early!

---

## 9. Prompt Engineering: Iterating on Goals and Behavior

**For LLM-enhanced agents, the prompt is a core iteration variable.**

Prompt = **Goal definition** + **Behavior constraints** + **Tool descriptions** + **Examples**

### The Prompt as Agent Configuration

Think of prompts like configuration files for the agent's "brain":

```typescript
// Traditional agent config
const agentConfig = {
  rules: [
    { condition: "failedLogins > 5", action: "block" }
  ],
  thresholds: { maxAttempts: 5 }
};

// LLM agent "config" (via prompt)
const agentPrompt = `
You are a security agent that protects against brute force attacks.

Goal: Prevent unauthorized access while minimizing false positives.

Rules:
- Block users after 5 failed login attempts within 5 minutes
- Challenge with MFA for suspicious but not conclusive activity
- Never block admin users

Available tools: blockUser, sendMFAChallenge, logEvent
`;
```

### Prompt Iteration Cycle

```
Draft Prompt → Test Behavior → Identify Issues → Refine Prompt → Repeat
```

**Example iteration:**

**Iteration 1: Vague**
```typescript
const prompt_v1 = "Protect against attacks";
// Problem: Agent doesn't know what "protect" means or how to do it
```

**Iteration 2: Goal Defined**
```typescript
const prompt_v2 = `
Goal: Detect and stop brute force attacks.
Success: Attacker is blocked and cannot authenticate.
`;
// Better: Goal is clear, but no guidance on how
```

**Iteration 3: Add Constraints**
```typescript
const prompt_v3 = `
Goal: Detect and stop brute force attacks.
Success: Attacker is blocked and cannot authenticate.

Constraints:
- Block after 5 failed attempts in 5 minutes
- Don't block legitimate users with typos
- Prefer MFA challenge over blocking when uncertain
`;
// Better: Now has decision criteria
```

**Iteration 4: Add Examples**
```typescript
const prompt_v4 = `
Goal: Detect and stop brute force attacks.
Success: Attacker is blocked and cannot authenticate.

Constraints:
- Block after 5 failed attempts in 5 minutes
- Don't block legitimate users with typos
- Prefer MFA challenge over blocking when uncertain

Examples:
1. User with 10 failed logins in 1 minute → Block
2. User with 3 failed logins over 30 minutes → Monitor only
3. User from new location with 1 failed login → MFA challenge
`;
// Best: Clear examples guide behavior
```

### Prompt Components

**1. Identity & Role**
```
You are a [type] agent that [purpose].
```

**2. Goal**
```
Goal: [What success looks like]
Success criteria: [Measurable conditions]
```

**3. Constraints & Rules**
```
Rules:
- [Must do X]
- [Must NOT do Y]
- [Prefer A over B when...]
```

**4. Tool Descriptions**
```
Available tools:
- blockUser(userId, duration): Prevents authentication for duration minutes
- sendMFAChallenge(userId): Sends 2FA code to user's phone
```

**5. Examples** (Few-shot learning)
```
Example scenarios:
1. Situation X → Action A because reason R
2. Situation Y → Action B because reason Q
```

**6. Output Format**
```
Your response must be:
{
  "decision": "block" | "challenge" | "allow",
  "reasoning": "why you made this decision",
  "confidence": 0-1
}
```

### Prompt Testing

Use evaluation harnesses to test prompts:

```typescript
// Test different prompts on same scenarios
const prompts = [prompt_v1, prompt_v2, prompt_v3, prompt_v4];

for (const prompt of prompts) {
  agent.setPrompt(prompt);
  const results = await harness.runEvaluation(agent, scenarios);
  console.log(`Prompt ${prompts.indexOf(prompt)}: ${passRate(results)}% pass rate`);
}

// Output:
// Prompt 0: 40% pass rate
// Prompt 1: 60% pass rate
// Prompt 2: 85% pass rate
// Prompt 3: 95% pass rate ← Winner!
```

### Common Prompt Problems

**Problem 1: Too Vague**
```
❌ "Improve system performance"
✅ "Reduce p95 API latency below 100ms by optimizing database queries"
```

**Problem 2: No Constraints**
```
❌ "Fix bugs in the code"
✅ "Fix bugs, but preserve existing function signatures and don't break tests"
```

**Problem 3: No Examples**
```
❌ "Use good judgment"
✅ "Example: If error rate > 5%, rollback. If 1-5%, alert. If < 1%, continue."
```

**Problem 4: Conflicting Instructions**
```
❌ "Be fast (make quick decisions) but thorough (analyze all possibilities)"
✅ "Prioritize speed over thoroughness. Make decisions within 2 seconds using available data only."
```

### Prompt Versioning

Track prompt changes like code:

```typescript
// prompts/auth-agent-v1.txt
const AUTH_AGENT_PROMPT_V1 = "...";

// prompts/auth-agent-v2.txt (improved)
const AUTH_AGENT_PROMPT_V2 = "...";

// Track which version is deployed
const CURRENT_PROMPT_VERSION = 'v2';
```

### Summary: Prompt Engineering

For LLM-enhanced agents:
- **Prompt = Agent brain configuration**
- **Iterate** on prompts like you iterate on code
- **Test** prompts with evaluation harnesses
- **Version** prompts and track changes
- **Components**: Identity, goal, constraints, tools, examples, output format

---

## 10. Debugging Agent Loops

When agents fail, you need visibility into the loop.

### Loop Tracing

```typescript
class AgentLoopTracer {
  async traceLoop(goalId: string): Promise<LoopTrace> {
    const trace = await this.db.query(`
      SELECT
        iteration,
        phase,  -- 'goal', 'plan', 'act', 'observe', 'reflect'
        input,
        output,
        duration_ms,
        timestamp
      FROM agent_loop_traces
      WHERE goal_id = $1
      ORDER BY iteration, timestamp
    `, [goalId]);

    return this.formatTrace(trace);
  }

  formatTrace(trace: RawTrace[]): LoopTrace {
    // Group by iteration
    const iterations = {};

    for (const event of trace) {
      if (!iterations[event.iteration]) {
        iterations[event.iteration] = {};
      }
      iterations[event.iteration][event.phase] = event;
    }

    return {
      goalId: trace[0].goal_id,
      totalIterations: Math.max(...trace.map(t => t.iteration)),
      success: trace[trace.length - 1].phase === 'success',
      iterations
    };
  }
}

// Example trace output:
{
  goalId: "goal-123",
  totalIterations: 3,
  success: true,
  iterations: {
    1: {
      goal: { input: "Block brute force attacker", successCriteria: "..." },
      plan: { output: "BLOCK_USER", reasoning: "5+ failed attempts" },
      act: { output: "User blocked for 60 min", duration_ms: 23 },
      observe: { output: "Failed logins stopped", duration_ms: 105 },
      reflect: { output: "Goal not achieved yet - monitor", duration_ms: 12 }
    },
    2: {
      plan: { output: "MONITOR", reasoning: "Wait for confirmation" },
      act: { output: "No new failed logins in 2 min", duration_ms: 120045 },
      observe: { output: "Attack stopped, user blocked", duration_ms: 89 },
      reflect: { output: "Goal achieved!", duration_ms: 8 }
    }
  }
}
```

---

## Summary

### Core Principles Checklist

When building an agent, ensure:

✅ **Loop is explicit**: Goal → Plan → Act → Observe → Reflect → Exit
✅ **Tools are well-designed**: Clear interfaces, safety constraints, fast feedback
✅ **Verification is built-in**: Automated checks or human approval
✅ **Start simple**: Level 2 first, add complexity incrementally
✅ **Metrics are tracked**: Loop efficiency, accuracy, cost, improvement

### Key Takeaways

1. **Agents operate in loops** - Make the loop explicit in your code
2. **Goals need verification** - How does agent know it succeeded?
3. **Tools enable action** - Design tools for safety and composability
4. **Start simple, evolve** - Don't build Level 4 on day one
5. **Measure effectiveness** - Track loop metrics, not just performance

### Next Steps

- Review your agent designs - do they have explicit loops?
- Audit your tools - are they well-specified and safe?
- Add loop metrics to your monitoring
- Plan incremental rollout (Level 2 → 3 → 4)

### Related Documents

- [PATTERNS.md](PATTERNS.md) - Core Agent Loop pattern implementation
- [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md) - Phase 2.5: Tools & Verification
- [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) - Levels 2-4 intelligence capabilities
- [README.md](README.md) - When to build an agent (updated criteria)
