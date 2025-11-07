# Multi-Agent Patterns

Guidance for evolving from single agents to multi-agent systems safely and intentionally.

## Overview

**Critical Principle**: Start with a single agent. Only introduce multiple agents when the benefits clearly outweigh the added complexity.

Multi-agent systems are Level 5 (Collaborative) in the agent taxonomy. They involve:
- Multiple autonomous agents working together
- Inter-agent communication and coordination
- Emergent behavior from agent interactions
- Shared goals or complementary specializations

---

## When to Introduce a Second Agent

**Don't introduce multiple agents prematurely!** The added complexity is significant.

### Triggers for Multi-Agent Systems

Introduce a second agent when you have **clear, independent concerns** that would benefit from specialization:

#### Trigger 1: **Conflicting Optimization Goals**

```typescript
// Problem: Single agent trying to optimize two conflicting goals
class DeploymentAgent {
  // Goal 1: Deploy fast
  // Goal 2: Deploy safely
  // → These conflict! Fast deploys = higher risk
}

// Solution: Two specialized agents
class FastDeployAgent {
  // Optimized for speed
  // Minimal checks, rapid rollout
}

class SafetyMonitorAgent {
  // Optimized for safety
  // Can halt fast deploy if issues detected
}
```

**When**: Single agent's goals conflict or pull in opposite directions

#### Trigger 2: **Distinct Expertise Domains**

```typescript
// Problem: Single agent needs expertise in multiple unrelated domains
class InfrastructureAgent {
  // Needs to know: Kubernetes, databases, networking, security, ...
  // → Too broad! Can't be expert in all
}

// Solution: Specialized agents per domain
class KubernetesAgent {
  // Expert in pod management, scaling, health checks
}

class DatabaseAgent {
  // Expert in query optimization, indexing, replication
}

class SecurityAgent {
  // Expert in authentication, authorization, threat detection
}
```

**When**: Problem spans multiple distinct technical domains

#### Trigger 3: **Different Operational Rhythms**

```typescript
// Problem: Single agent handling both real-time and batch operations
class MonitoringAgent {
  // Real-time: Check metrics every second
  // Batch: Generate reports daily
  // → Different timescales make single agent awkward
}

// Solution: Separate agents for different rhythms
class RealTimeMonitorAgent {
  // Runs continuously, sub-second loops
  // Detects anomalies, sends alerts
}

class ReportingAgent {
  // Runs daily
  // Aggregates data, generates insights
}
```

**When**: Operations have fundamentally different timescales or frequencies

#### Trigger 4: **Independent Failure Domains**

```typescript
// Problem: Single agent failure takes down multiple functions
class MasterAgent {
  // Handles auth, logging, metrics, alerting
  // → If it crashes, everything stops
}

// Solution: Independent agents for resilience
class AuthAgent {
  // Critical: Must never fail
  // Isolated, simple, highly available
}

class ObservabilityAgent {
  // Non-critical: Can fail without breaking app
  // Separate failure domain
}
```

**When**: Failure isolation is critical for reliability

### Decision Tree: Should You Add a Second Agent?

```
Question 1: Does the new functionality conflict with existing agent's goals?
  → YES: Consider second agent
  → NO: Add to existing agent

Question 2: Is the new domain completely unrelated to existing agent's expertise?
  → YES: Consider second agent
  → NO: Add to existing agent

Question 3: Does the new functionality have a different operational rhythm?
  → YES: Consider second agent
  → NO: Add to existing agent

Question 4: Would failure of the new functionality bring down critical paths?
  → YES: Consider second agent
  → NO: Add to existing agent

If ANY answer is "Consider second agent":
  → Evaluate if benefits > complexity cost
  → Start with single agent, split only when pain is clear
```

---

## Multi-Agent Communication Patterns

When you have multiple agents, they need to communicate. Choose the right pattern:

### Pattern 1: **Fire-and-Forget Events**

**When**: Agents notify others of state changes, but don't need acknowledgment.

```typescript
// Observer agent publishes events, doesn't wait for response
class MetricsAgent {
  async recordMetric(metric: Metric) {
    await this.save(metric);

    // Publish event (fire-and-forget)
    await this.eventBus.publish({
      type: 'metric.recorded',
      data: metric
    });
    // Don't wait for other agents to process
  }
}

// Alerting agent subscribes to events
class AlertingAgent {
  constructor() {
    this.eventBus.subscribe('metric.recorded', this.handleMetric);
  }

  async handleMetric(metric: Metric) {
    if (metric.value > threshold) {
      await this.sendAlert(metric);
    }
  }
}
```

**Pros:**
- Loose coupling
- Non-blocking
- Easy to add new subscribers

**Cons:**
- No guarantee of processing
- Eventual consistency
- Hard to debug flows

**Use when**: Events are informational, not critical

### Pattern 2: **Request-Response RPC**

**When**: One agent needs a decision or data from another and must wait.

```typescript
// Security agent requests risk assessment from ML agent
class SecurityAgent {
  async handleLogin(userId: string) {
    // Request risk score from specialized ML agent
    const riskScore = await this.mlAgent.assessRisk({
      userId,
      location: currentLocation,
      device: currentDevice
    });

    if (riskScore > 0.8) {
      await this.blockUser(userId);
    }
  }
}

// ML agent provides risk scoring service
class MLRiskAgent {
  async assessRisk(context: RiskContext): Promise<number> {
    const features = this.extractFeatures(context);
    const score = await this.model.predict(features);
    return score;
  }
}
```

**Pros:**
- Synchronous, predictable
- Clear dependencies
- Easy to reason about

**Cons:**
- Tight coupling
- Blocking (latency sensitive)
- Single point of failure

**Use when**: Decision must be synchronous

### Pattern 3: **Shared State (Coordination)**

**When**: Multiple agents need to coordinate on shared resources or goals.

```typescript
// Deployment coordination via shared state
class DeploymentCoordinator {
  sharedState: {
    currentDeployment: DeploymentInfo | null;
    lock: boolean;
  };

  async acquireLock(agentId: string): Promise<boolean> {
    if (this.sharedState.lock) {
      return false; // Another agent is deploying
    }
    this.sharedState.lock = true;
    this.sharedState.currentDeployment = { agentId, startTime: Date.now() };
    return true;
  }

  async releaseLock(agentId: string) {
    if (this.sharedState.currentDeployment?.agentId === agentId) {
      this.sharedState.lock = false;
      this.sharedState.currentDeployment = null;
    }
  }
}

// Agents coordinate via shared state
class CanaryDeployAgent {
  async deploy(version: string) {
    const acquired = await coordinator.acquireLock(this.id);
    if (!acquired) {
      this.log('Another deployment in progress, waiting...');
      return;
    }

    try {
      await this.performDeploy(version);
    } finally {
      await coordinator.releaseLock(this.id);
    }
  }
}
```

**Pros:**
- Prevents conflicts
- Enables coordination
- Supports complex workflows

**Cons:**
- Shared mutable state (complexity)
- Requires consensus/locking
- Potential deadlocks

**Use when**: Agents must coordinate on exclusive resources

### Pattern 4: **Hierarchical (Orchestrator + Workers)**

**When**: One agent orchestrates, others execute specialized tasks.

```typescript
// Orchestrator agent delegates to specialized workers
class InfrastructureOrchestratorAgent {
  workers: {
    kubernetes: KubernetesAgent;
    database: DatabaseAgent;
    network: NetworkAgent;
  };

  async deployFullStack(config: StackConfig) {
    // Goal: Deploy entire infrastructure

    // Iteration 1: Deploy database first (dependency)
    const dbResult = await this.workers.database.deploy(config.db);

    // Iteration 2: Set up networking
    const networkResult = await this.workers.network.configure(config.network);

    // Iteration 3: Deploy K8s apps
    const k8sResult = await this.workers.kubernetes.deploy({
      ...config.k8s,
      dbEndpoint: dbResult.endpoint
    });

    return {
      database: dbResult,
      network: networkResult,
      kubernetes: k8sResult
    };
  }
}
```

**Pros:**
- Clear responsibility hierarchy
- Orchestrator has full visibility
- Easy to reason about flows

**Cons:**
- Single point of failure (orchestrator)
- Orchestrator can become complex
- Workers less autonomous

**Use when**: Complex workflows with dependencies

### Pattern 5: **Peer-to-Peer (Collaborative)**

**When**: Agents negotiate and collaborate as equals.

```typescript
// Agents negotiate optimal resource allocation
class ResourceAgent {
  async negotiate(peers: ResourceAgent[]) {
    // Each agent proposes its desired resources
    const proposals = await Promise.all(
      peers.map(peer => peer.proposeAllocation())
    );

    // Agents vote on allocations
    const votes = await Promise.all(
      peers.map(peer => peer.vote(proposals))
    );

    // Consensus: Highest voted proposal wins
    const winner = this.selectConsensus(votes);
    return winner;
  }

  async proposeAllocation(): Promise<Proposal> {
    return {
      agentId: this.id,
      requestedCPU: this.calculateNeeds(),
      priority: this.priority
    };
  }

  async vote(proposals: Proposal[]): Promise<Vote> {
    // Vote based on fairness, priority, system capacity
    return proposals.map(p => ({
      proposalId: p.id,
      score: this.evaluateFairness(p)
    }));
  }
}
```

**Pros:**
- No single point of failure
- Democratic/fair decisions
- Highly autonomous

**Cons:**
- Complex to implement
- Consensus is hard
- Unpredictable behavior

**Use when**: No natural hierarchy exists

---

## Self-Improving Tools

Tools that learn and adapt their behavior based on outcomes.

### Pattern: Tool with Feedback Loop

```typescript
class SelfImprovingBlockUserTool {
  // Tool maintains its own performance history
  history: {
    blockId: string;
    wasEffective: boolean;
    falsePositive: boolean;
    context: any;
  }[] = [];

  async blockUser(userId: string, context: any): Promise<BlockResult> {
    // Execute the block
    const blockId = await this.performBlock(userId);

    // Record context for later learning
    this.history.push({
      blockId,
      wasEffective: null,  // Unknown at block time
      falsePositive: null,
      context
    });

    return { blockId, success: true };
  }

  // Feedback: Was the block effective?
  async recordOutcome(blockId: string, outcome: Outcome) {
    const entry = this.history.find(h => h.blockId === blockId);
    if (entry) {
      entry.wasEffective = outcome.attackStopped;
      entry.falsePositive = outcome.legitimateUser;

      // Learn from outcome
      await this.updateModel(entry);
    }
  }

  async updateModel(entry: HistoryEntry) {
    // Adjust thresholds based on outcomes
    if (entry.falsePositive) {
      // Too aggressive, increase threshold
      this.blockThreshold += 0.05;
    } else if (!entry.wasEffective) {
      // Too lenient, decrease threshold
      this.blockThreshold -= 0.05;
    }

    // Or train ML model
    await this.mlModel.update({
      features: entry.context,
      label: entry.wasEffective ? 1 : 0
    });
  }

  // Tool improves its own decision-making
  async shouldBlock(context: any): Promise<boolean> {
    const riskScore = await this.calculateRisk(context);

    // Use learned threshold (not static)
    return riskScore > this.blockThreshold;
  }
}
```

### Self-Improving Tool Checklist

- [ ] **Track outcomes**: Record what happened after tool use
- [ ] **Feedback mechanism**: How does tool learn if it was correct?
- [ ] **Update logic**: How does tool adjust based on feedback?
- [ ] **Bounded adaptation**: Prevent tool from drifting too far
- [ ] **Rollback capability**: Can revert if improvements backfire
- [ ] **Monitoring**: Track tool effectiveness over time

### Example: Self-Improving Deployment Tool

```typescript
class SelfImprovingDeployTool {
  // Learns optimal rollout percentages
  rolloutStrategy = {
    initialPercentage: 10,
    incrementPercentage: 10,
    waitTime: 120  // seconds
  };

  async deploy(version: string): Promise<DeployResult> {
    let deployed = 0;

    while (deployed < 100) {
      const nextPercentage = Math.min(
        deployed + this.rolloutStrategy.incrementPercentage,
        100
      );

      await this.deployToPercentage(version, nextPercentage);
      deployed = nextPercentage;

      await this.wait(this.rolloutStrategy.waitTime);

      const metrics = await this.observeMetrics();

      if (metrics.errorRate > 0.01) {
        await this.rollback(version);
        await this.recordOutcome({ success: false, reason: 'high-error-rate' });
        return { success: false };
      }
    }

    await this.recordOutcome({ success: true });
    return { success: true };
  }

  async recordOutcome(outcome: DeployOutcome) {
    if (outcome.success) {
      // Deployment succeeded, can we go faster next time?
      this.rolloutStrategy.incrementPercentage = Math.min(
        this.rolloutStrategy.incrementPercentage + 5,
        50  // Max 50% increments
      );
      this.rolloutStrategy.waitTime = Math.max(
        this.rolloutStrategy.waitTime - 10,
        60  // Min 60 seconds wait
      );
    } else {
      // Deployment failed, go slower next time
      this.rolloutStrategy.incrementPercentage = Math.max(
        this.rolloutStrategy.incrementPercentage - 5,
        5  // Min 5% increments
      );
      this.rolloutStrategy.waitTime = Math.min(
        this.rolloutStrategy.waitTime + 30,
        300  // Max 5 minutes wait
      );
    }
  }
}
```

---

## Safety Considerations for Multi-Agent Systems

Multi-agent systems introduce new failure modes. Design for safety from the start.

### Safety Pattern 1: **Agent Isolation**

Agents should fail independently.

```typescript
// Bad: Agents share resources, cascading failures
class SharedResourceAgents {
  sharedDatabase: Database;  // If DB fails, all agents fail
  sharedCache: Redis;        // If cache fails, all agents slow down
}

// Good: Agents have isolated resources
class IsolatedAgents {
  agent1: {
    database: Database,  // Separate DB instance
    cache: LocalCache    // Local cache, no cross-agent deps
  };

  agent2: {
    database: Database,  // Different DB instance
    cache: LocalCache
  };
}
```

### Safety Pattern 2: **Conflict Resolution**

When agents disagree, have clear resolution rules.

```typescript
class SecurityDecisionResolver {
  // Multiple security agents vote on action
  async resolveConflict(decisions: Decision[]): Promise<Action> {
    const votes = {
      block: decisions.filter(d => d.action === 'block').length,
      allow: decisions.filter(d => d.action === 'allow').length,
      challenge: decisions.filter(d => d.action === 'challenge').length
    };

    // Safety bias: Most restrictive action wins
    if (votes.block > 0) {
      return 'block';  // Any agent voting block → block
    } else if (votes.challenge > 0) {
      return 'challenge';  // Challenge if no blocks but at least one challenge
    } else {
      return 'allow';  // Only allow if all agents agree
    }
  }
}
```

### Safety Pattern 3: **Rate Limiting Inter-Agent Communication**

Prevent agents from overwhelming each other.

```typescript
class RateLimitedCommunication {
  // Each agent can only send N requests per second to other agents
  limits = new Map<string, RateLimiter>();

  async sendToAgent(fromAgent: string, toAgent: string, message: any) {
    const limiter = this.limits.get(`${fromAgent}->${toAgent}`);

    if (!limiter.allow()) {
      throw new Error(`Rate limit exceeded: ${fromAgent} → ${toAgent}`);
    }

    await this.deliver(toAgent, message);
  }
}
```

### Safety Pattern 4: **Circuit Breaker for Agent Calls**

If an agent is failing, stop calling it.

```typescript
class AgentCircuitBreaker {
  failures = new Map<string, number>();

  async callAgent(agentId: string, request: any): Promise<any> {
    const failureCount = this.failures.get(agentId) || 0;

    if (failureCount > 5) {
      // Circuit open: Agent is failing, don't call
      throw new Error(`Agent ${agentId} circuit is open (too many failures)`);
    }

    try {
      const result = await this.invoke(agentId, request);
      this.failures.set(agentId, 0);  // Success, reset counter
      return result;
    } catch (error) {
      this.failures.set(agentId, failureCount + 1);
      throw error;
    }
  }
}
```

### Safety Pattern 5: **Timeout for Multi-Agent Operations**

Don't let multi-agent coordination hang forever.

```typescript
class MultiAgentOrchestrator {
  async coordinateAgents(agents: Agent[], goal: Goal): Promise<Result> {
    const timeout = 30000;  // 30 seconds max

    const result = await Promise.race([
      this.runCoordination(agents, goal),
      this.timeoutPromise(timeout)
    ]);

    if (result === 'TIMEOUT') {
      // Coordination failed, fall back to safe default
      return this.fallbackStrategy(goal);
    }

    return result;
  }
}
```

---

## Evolution Path: Single → Multi-Agent

**Recommended timeline:**

### Month 1-3: **Single Agent**
- Build one agent well
- Get it to production
- Learn its failure modes
- Validate the concept

### Month 4-6: **Evaluate Multi-Agent Need**
- Are goals conflicting?
- Are domains distinct?
- Is specialization needed?
- **If NO to all → Stay single agent**

### Month 7+: **Introduce Second Agent (If Justified)**
- Start with simplest pattern (fire-and-forget events)
- Add isolation and safety patterns from day one
- Monitor interaction overhead
- Evaluate if added complexity is worth it

### Later: **Add More Agents (Only If Clear Value)**
- Each new agent adds complexity exponentially
- Justify each addition with clear ROI
- Consider if single agent with better tools would suffice

---

## Multi-Agent Anti-Patterns

### Anti-Pattern 1: **Premature Multi-Agent**

```
❌ "We might need multiple agents in the future, let's build for it now"
✅ "Start with one agent. Split when pain is clear."
```

### Anti-Pattern 2: **Agents Calling Agents Calling Agents**

```
❌ Agent A → Agent B → Agent C → Agent D (deep chains)
✅ Orchestrator → [Agent A, Agent B, Agent C] (flat hierarchy)
```

### Anti-Pattern 3: **Chatty Agents**

```
❌ Agents send 100 messages per second to each other
✅ Agents batch updates, communicate only when necessary
```

### Anti-Pattern 4: **Agents Without Clear Boundaries**

```
❌ "Agent A sometimes does what Agent B does"
✅ "Agent A owns domain X, Agent B owns domain Y, clear separation"
```

### Anti-Pattern 5: **No Conflict Resolution**

```
❌ Two agents make contradictory decisions, system stuck
✅ Clear resolution protocol: most restrictive wins, orchestrator decides, etc.
```

---

## Summary

### When to Use Multiple Agents

✅ **Do introduce multiple agents when:**
- Goals fundamentally conflict
- Domains are completely distinct
- Operational rhythms differ significantly
- Failure isolation is critical

❌ **Don't introduce multiple agents when:**
- Single agent can handle it (even if complex)
- Separation of concerns can be achieved with better tools
- Communication overhead > specialization benefit
- "It might be useful later" (YAGNI)

### Communication Patterns

Choose based on coupling requirements:
- **Fire-and-forget**: Loose coupling, eventual consistency
- **Request-response**: Tight coupling, synchronous
- **Shared state**: Coordination, exclusive resources
- **Hierarchical**: Clear control, orchestrated workflows
- **Peer-to-peer**: Democratic, no single point of failure

### Safety First

- Isolate agent failures
- Define conflict resolution upfront
- Rate limit inter-agent communication
- Use circuit breakers
- Set timeouts on coordination
- Monitor agent interaction overhead

### Evolution Path

Start simple → Validate need → Add complexity intentionally → Measure value

**Remember**: Most problems don't need multiple agents. A single well-designed agent with good tools is often the right answer.
