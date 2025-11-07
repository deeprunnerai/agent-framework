# Agent Patterns

Reusable architectural patterns extracted from production agents (costing-agent, auth-agent) and industry best practices.

## Pattern Categories

1. [Core Agent Loop Pattern](#core-agent-loop-pattern)
2. [Lifecycle Patterns](#lifecycle-patterns)
3. [Communication Patterns](#communication-patterns)
4. [Resilience Patterns](#resilience-patterns)
5. [Instrumentation Patterns](#instrumentation-patterns)
6. [Multi-Tenancy Patterns](#multi-tenancy-patterns)
7. [Caching Patterns](#caching-patterns)
8. [Intelligence Patterns](#intelligence-patterns)

---

## Core Agent Loop Pattern

**Problem**: Agents need a structured way to pursue goals autonomously, with clear iteration and exit conditions.

**Solution**: Implement an explicit Goal → Plan → Act → Observe → Reflect loop with measurable success criteria.

### Pattern Structure

```
┌──────────────────────────────────────┐
│         AGENT LOOP PATTERN           │
└──────────────────────────────────────┘

GOAL      → Define what success looks like
  ↓
PLAN      → Break goal into actionable steps
  ↓
ACT       → Execute tools/actions
  ↓
OBSERVE   → Collect feedback/results
  ↓
REFLECT   → Analyze: did it work? adjust?
  ↓
EXIT or LOOP BACK
```

### Implementation (TypeScript)

```typescript
interface Goal {
  description: string;
  successCriteria: (observation: any) => boolean;
  maxIterations: number;
}

interface LoopResult {
  success: boolean;
  iterations: number;
  finalState: any;
  reason?: string;
}

class AgentLoop {
  async pursue(goal: Goal): Promise<LoopResult> {
    let iteration = 0;
    let currentState = await this.getInitialState();

    while (iteration < goal.maxIterations) {
      iteration++;

      // PLAN: Decide next action
      const plan = await this.plan(goal, currentState);

      if (plan.shouldExit) {
        return {
          success: false,
          iterations: iteration,
          finalState: currentState,
          reason: plan.exitReason
        };
      }

      // ACT: Execute the plan
      const actionResult = await this.act(plan);

      // OBSERVE: Collect new state
      const observation = await this.observe(actionResult);

      // REFLECT: Check if goal achieved
      const goalAchieved = goal.successCriteria(observation);

      if (goalAchieved) {
        // Success - log and exit
        await this.logSuccess(goal, iteration);
        return {
          success: true,
          iterations: iteration,
          finalState: observation
        };
      }

      // Update state for next iteration
      currentState = observation;

      // Adjust strategy based on feedback
      await this.adjustStrategy(observation);
    }

    // Max iterations reached without success
    await this.logFailure(goal, iteration);
    return {
      success: false,
      iterations: iteration,
      finalState: currentState,
      reason: 'Max iterations reached'
    };
  }

  async plan(goal: Goal, state: any): Promise<Plan> {
    // Determine what action to take based on current state
    // This is where agent intelligence goes
  }

  async act(plan: Plan): Promise<ActionResult> {
    // Execute the planned action using available tools
  }

  async observe(actionResult: ActionResult): Promise<Observation> {
    // Collect feedback from the action
    // Run tests, check metrics, query state
  }

  async adjustStrategy(observation: Observation): Promise<void> {
    // Learn from this iteration
    // Update internal models, adjust thresholds
  }
}
```

### Example: Deployment Agent

```typescript
const deploymentGoal: Goal = {
  description: "Deploy v2.0 with < 0.1% error rate",
  successCriteria: (obs) =>
    obs.deployedPercentage === 100 &&
    obs.errorRate < 0.001,
  maxIterations: 10
};

const result = await deploymentAgent.pursue(deploymentGoal);

if (result.success) {
  console.log(`Deployed successfully in ${result.iterations} iterations`);
} else {
  console.log(`Deployment failed: ${result.reason}`);
  await rollback();
}
```

### Example: Auth Agent - Threat Response

```typescript
const threatGoal: Goal = {
  description: "Stop brute force attack on user-123",
  successCriteria: (obs) =>
    obs.failedLogins < 5 &&
    obs.attackerBlocked &&
    obs.timeSinceLastAttempt > 300, // 5 minutes
  maxIterations: 5
};

class AuthAgentLoop extends AgentLoop {
  async plan(goal: Goal, state: any): Promise<Plan> {
    const context = state;

    // Escalating response strategy
    if (context.failedLogins >= 10) {
      return { action: 'BLOCK_IP', severity: 'high' };
    } else if (context.failedLogins >= 5) {
      return { action: 'BLOCK_USER', severity: 'medium' };
    } else {
      return { action: 'MONITOR', severity: 'low' };
    }
  }

  async act(plan: Plan): Promise<ActionResult> {
    switch (plan.action) {
      case 'BLOCK_IP':
        await this.blockIP(this.attackerIP);
        return { blocked: 'ip', success: true };
      case 'BLOCK_USER':
        await this.blockUser(this.userId);
        return { blocked: 'user', success: true };
      case 'MONITOR':
        await sleep(60000); // Wait 1 minute
        return { monitored: true };
    }
  }

  async observe(actionResult: ActionResult): Promise<Observation> {
    return {
      failedLogins: await this.getFailedLoginCount(),
      attackerBlocked: actionResult.blocked !== undefined,
      timeSinceLastAttempt: await this.getTimeSinceLastAttempt(),
      attackPersists: await this.isAttackOngoing()
    };
  }
}
```

### Metrics to Track

```typescript
interface LoopMetrics {
  // Efficiency
  avgIterationsToGoal: number;      // Target: < 5
  convergenceRate: number;           // % of goals achieved (target: > 95%)
  maxIterationsReachedCount: number; // How often we hit the limit

  // Cost
  avgCostPerGoal: number;           // $ spent per successful goal
  toolInvocationsPerGoal: number;   // Tool calls per goal
}
```

### When to Use This Pattern

✅ Use when:
- Agent needs to pursue goals autonomously
- Number of steps is uncertain
- Need measurable progress and exit conditions
- Want debuggable agent behavior

❌ Don't use when:
- Fixed sequence of operations (use workflow)
- Single action required (use simple function)
- No feedback available to verify progress

### Benefits

- **Measurable**: Track iterations, convergence rate
- **Debuggable**: Clear loop trace for failures
- **Improvable**: Optimize loop efficiency over time
- **Explainable**: Each iteration has clear reasoning

See [CORE_AGENT_MECHANICS.md](CORE_AGENT_MECHANICS.md) for comprehensive details on agent loops, tools, and verification.

---

## Lifecycle Patterns

### Pattern 1: Auto-Registration

**Problem**: Agent needs to announce itself and get configuration

**Solution**: Self-register on initialization with deployment metadata

**Implementation** (TypeScript):
```typescript
class Agent {
  async init(config: Config): Promise<void> {
    // 1. Generate unique fingerprint
    const fingerprint = await this.generateFingerprint();

    // 2. Register with service
    const response = await this.client.post('/register', {
      projectId: config.projectId,
      fingerprint,
      environment: process.env.NODE_ENV || 'development',
      platform: 'nodejs',
      version: process.version,
      hostname: os.hostname(),
      metadata: config.metadata
    });

    // 3. Store deployment ID
    this.deploymentId = response.data.deployment_id;
    this.config = { ...config, ...response.data.server_config };

    // 4. Start heartbeat
    this.startHeartbeat();
  }

  private async generateFingerprint(): Promise<string> {
    // Use stable machine characteristics
    const interfaces = os.networkInterfaces();
    const cpus = os.cpus();
    const data = JSON.stringify({ interfaces, cpus, hostname: os.hostname() });
    return crypto.createHash('sha256').update(data).digest('hex');
  }
}
```

**Used In**: costing-agent, recommended for all agents

---

### Pattern 2: Graceful Shutdown

**Problem**: Agent needs to cleanup on process exit

**Solution**: Register signal handlers, flush buffers, close connections

**Implementation**:
```typescript
class Agent {
  async shutdown(): Promise<void> {
    console.log('Agent shutting down...');

    // 1. Stop accepting new data
    this.accepting = false;

    // 2. Flush pending data
    await this.buffer.flush();

    // 3. Stop heartbeat
    this.stopHeartbeat();

    // 4. Close connections
    await this.client.close();

    console.log('Agent shutdown complete');
  }

  setupSignalHandlers(): void {
    const signals: NodeJS.Signals[] = ['SIGINT', 'SIGTERM', 'SIGQUIT'];

    signals.forEach(signal => {
      process.on(signal, async () => {
        await this.shutdown();
        process.exit(0);
      });
    });

    process.on('uncaughtException', async (error) => {
      console.error('Uncaught exception:', error);
      await this.shutdown();
      process.exit(1);
    });
  }
}
```

**Used In**: costing-agent, all agents

---

### Pattern 3: Heartbeat

**Problem**: Service needs to know agent is alive

**Solution**: Periodic heartbeat to update last_seen timestamp

**Implementation**:
```typescript
class Agent {
  private heartbeatInterval?: NodeJS.Timeout;

  startHeartbeat(intervalMs: number = 60000): void {
    this.heartbeatInterval = setInterval(async () => {
      try {
        await this.client.post(`/deployments/${this.deploymentId}/heartbeat`, {
          timestamp: new Date().toISOString(),
          status: 'active',
          metrics: this.collectMetrics()
        });
      } catch (error) {
        console.error('Heartbeat failed:', error);
        // Don't crash on heartbeat failure
      }
    }, intervalMs);
  }

  stopHeartbeat(): void {
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
    }
  }
}
```

**Used In**: costing-agent

---

## Communication Patterns

### Pattern 4: Event Buffer

**Problem**: Sending individual events is inefficient

**Solution**: Batch events with dual triggers (size + time)

**Implementation**:
```typescript
class EventBuffer<T> {
  private buffer: T[] = [];
  private flushTimer?: NodeJS.Timeout;

  constructor(
    private maxSize: number = 100,
    private maxWaitMs: number = 10000,
    private onFlush: (events: T[]) => Promise<void>
  ) {}

  add(event: T): void {
    this.buffer.push(event);

    // Size trigger
    if (this.buffer.length >= this.maxSize) {
      this.flush();
    }

    // Start time trigger if not already running
    if (!this.flushTimer) {
      this.flushTimer = setTimeout(() => this.flush(), this.maxWaitMs);
    }
  }

  async flush(): Promise<void> {
    if (this.buffer.length === 0) return;

    // Clear timer
    if (this.flushTimer) {
      clearTimeout(this.flushTimer);
      this.flushTimer = undefined;
    }

    // Take events
    const events = this.buffer.splice(0);

    // Send
    try {
      await this.onFlush(events);
    } catch (error) {
      console.error('Flush failed:', error);
      // Could implement retry or dead letter queue here
    }
  }
}
```

**Used In**: costing-agent

---

### Pattern 5: Resilient Transmitter

**Problem**: Network failures shouldn't crash agent

**Solution**: Retry with exponential backoff + circuit breaker

**Implementation**:
```typescript
class ResilientTransmitter {
  private failureCount = 0;
  private circuitOpen = false;
  private circuitOpenUntil = 0;

  constructor(
    private maxRetries: number = 3,
    private baseDelayMs: number = 1000,
    private circuitBreakerThreshold: number = 5,
    private circuitBreakerResetMs: number = 60000
  ) {}

  async send(url: string, data: any): Promise<void> {
    // Check circuit breaker
    if (this.circuitOpen) {
      if (Date.now() < this.circuitOpenUntil) {
        throw new Error('Circuit breaker open');
      }
      // Try to close circuit
      this.circuitOpen = false;
      this.failureCount = 0;
    }

    let lastError: Error | undefined;

    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        await this.client.post(url, data, { timeout: 5000 });

        // Success - reset failure count
        this.failureCount = 0;
        return;

      } catch (error) {
        lastError = error as Error;

        if (attempt < this.maxRetries) {
          // Exponential backoff
          const delay = this.baseDelayMs * Math.pow(2, attempt);
          await this.sleep(Math.min(delay, 30000));
        }
      }
    }

    // All retries failed
    this.failureCount++;

    // Open circuit breaker if threshold exceeded
    if (this.failureCount >= this.circuitBreakerThreshold) {
      this.circuitOpen = true;
      this.circuitOpenUntil = Date.now() + this.circuitBreakerResetMs;
      console.error('Circuit breaker opened');
    }

    throw lastError || new Error('Send failed');
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

**Used In**: costing-agent, recommended for all agents

---

### Pattern 6: Request Compression

**Problem**: Large payloads increase network cost and latency

**Solution**: Compress payloads with gzip

**Implementation**:
```typescript
import zlib from 'zlib';
import { promisify } from 'util';

const gzip = promisify(zlib.gzip);

class CompressedTransmitter {
  async send(url: string, data: any): Promise<void> {
    const json = JSON.stringify(data);

    // Only compress if payload is large enough
    if (json.length > 1024) {
      const compressed = await gzip(json);

      await this.client.post(url, compressed, {
        headers: {
          'Content-Type': 'application/json',
          'Content-Encoding': 'gzip'
        }
      });
    } else {
      await this.client.post(url, data);
    }
  }
}
```

**Used In**: costing-agent

---

## Resilience Patterns

### Pattern 7: Graceful Degradation

**Problem**: Agent failure shouldn't break application

**Solution**: Try-catch wrappers, fail silently with logging

**Implementation**:
```typescript
class Agent {
  track(event: Event): void {
    try {
      this.buffer.add(event);
    } catch (error) {
      // Log error but don't throw
      console.error('Failed to track event:', error);
      // Optionally: emit metric for monitoring
      this.metrics.increment('track_errors');
    }
  }

  async validate(token: string): Promise<User | null> {
    try {
      return await this.doValidate(token);
    } catch (error) {
      console.error('Validation failed:', error);

      // For controller agents, decide: fail-open or fail-closed?
      if (this.config.failOpen) {
        return null; // Allow access on error
      } else {
        throw error; // Deny access on error
      }
    }
  }
}
```

**Used In**: costing-agent, auth-agent

---

### Pattern 8: Health Checks

**Problem**: Need to monitor agent health

**Solution**: Expose health endpoint with detailed status

**Implementation**:
```typescript
class Agent {
  getStatus(): HealthStatus {
    return {
      status: this.isHealthy() ? 'healthy' : 'unhealthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      components: {
        buffer: {
          status: this.buffer.size < this.buffer.maxSize ? 'ok' : 'full',
          size: this.buffer.size,
          capacity: this.buffer.maxSize
        },
        transmitter: {
          status: this.transmitter.circuitOpen ? 'degraded' : 'ok',
          failureCount: this.transmitter.failureCount
        },
        service: {
          status: this.lastHeartbeatSuccess ? 'connected' : 'disconnected',
          lastHeartbeat: this.lastHeartbeatTime
        }
      }
    };
  }

  isHealthy(): boolean {
    return !this.transmitter.circuitOpen &&
           this.buffer.size < this.buffer.maxSize &&
           this.lastHeartbeatSuccess;
  }
}
```

**Used In**: All agents

---

## Instrumentation Patterns

### Pattern 9: Auto-Instrumentation (Monkey Patching)

**Problem**: Don't want to manually wrap every SDK call

**Solution**: Intercept SDK at module level

**Implementation**:
```typescript
// auto.js - Entry point for auto-instrumentation
import { CostingAgent } from './index';

// Instrument OpenAI
try {
  const openaiModule = require.cache[require.resolve('openai')];
  if (openaiModule) {
    instrumentOpenAI(openaiModule.exports);
  }
} catch (e) {
  // OpenAI not installed, skip
}

function instrumentOpenAI(openaiExports: any): void {
  const OriginalOpenAI = openaiExports.OpenAI;

  openaiExports.OpenAI = class InstrumentedOpenAI extends OriginalOpenAI {
    constructor(...args: any[]) {
      super(...args);

      // Wrap methods
      const originalCreate = this.chat.completions.create.bind(this.chat.completions);
      this.chat.completions.create = async (params: any) => {
        const startTime = Date.now();

        try {
          const result = await originalCreate(params);

          // Track success
          agent.track({
            type: 'llm_call',
            provider: 'openai',
            model: params.model,
            tokens: result.usage?.total_tokens,
            latency: Date.now() - startTime,
            success: true
          });

          return result;
        } catch (error) {
          // Track error
          agent.track({
            type: 'llm_call',
            provider: 'openai',
            model: params.model,
            latency: Date.now() - startTime,
            success: false,
            error: error.message
          });

          throw error;
        }
      };
    }
  };
}
```

**Used In**: costing-agent

**Usage**:
```typescript
// User's app - just add at top of entry file
require('@deeprunnerai/costing-agent/auto');

// Rest of app code - OpenAI now auto-tracked
const openai = new OpenAI({ apiKey: '...' });
await openai.chat.completions.create({ ... }); // Automatically tracked!
```

---

### Pattern 10: Middleware (Express)

**Problem**: Need to protect routes without manual calls

**Solution**: Express middleware pattern

**Implementation**:
```typescript
class AuthAgent {
  middleware = {
    requireAuth: () => {
      return async (req: Request, res: Response, next: NextFunction) => {
        try {
          // Extract token
          const token = this.extractToken(req);

          if (!token) {
            return res.status(401).json({ error: 'No token provided' });
          }

          // Validate (with caching)
          const user = await this.validateToken(token);

          // Attach to request
          req.user = user;

          next();
        } catch (error) {
          return res.status(401).json({ error: 'Invalid token' });
        }
      };
    },

    optionalAuth: () => {
      return async (req: Request, res: Response, next: NextFunction) => {
        try {
          const token = this.extractToken(req);
          if (token) {
            req.user = await this.validateToken(token);
          }
        } catch (error) {
          // Ignore errors for optional auth
        }
        next();
      };
    }
  };
}
```

**Used In**: auth-agent

**Usage**:
```typescript
router.get('/protected',
  auth.middleware.requireAuth(),
  (req, res) => {
    res.json({ user: req.user });
  }
);
```

---

### Pattern 11: Dependency Injection (FastAPI)

**Problem**: FastAPI uses dependency injection pattern

**Solution**: Create dependency functions

**Implementation**:
```python
from fastapi import Depends, HTTPException, Header
from typing import Optional

class AuthAgent:
    def get_token_from_header(
        self,
        authorization: Optional[str] = Header(None)
    ) -> str:
        """Extract token from Authorization header"""
        if not authorization:
            raise HTTPException(status_code=401, detail="No token provided")

        if not authorization.startswith("Bearer "):
            raise HTTPException(status_code=401, detail="Invalid token format")

        return authorization[7:]  # Remove "Bearer " prefix

    async def get_current_user(
        self,
        token: str = Depends(get_token_from_header)
    ) -> User:
        """Validate token and return user"""
        try:
            user = await self.validate_token(token)
            return user
        except Exception as e:
            raise HTTPException(status_code=401, detail="Invalid token")

# Usage
agent = AuthAgent()

@app.get("/protected")
async def protected_route(user: User = Depends(agent.get_current_user)):
    return {"user": user}
```

**Used In**: auth-agent (Python SDK)

---

## Multi-Tenancy Patterns

### Pattern 12: Hierarchical Tenancy

**Problem**: Need organization isolation with project grouping

**Solution**: Three-level hierarchy

**Database Schema**:
```sql
-- Level 1: Organization (auth boundary)
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR NOT NULL,
  api_key_hash VARCHAR UNIQUE NOT NULL,
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Level 2: Projects (logical grouping)
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  name VARCHAR NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(organization_id, name)
);

-- Level 3: Deployments (physical instances)
CREATE TABLE deployments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id),
  fingerprint VARCHAR UNIQUE NOT NULL,
  environment VARCHAR,  -- dev, staging, prod
  status VARCHAR,       -- active, inactive
  last_seen_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Data scoped to deployment
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deployment_id UUID REFERENCES deployments(id),
  event_type VARCHAR NOT NULL,
  data JSONB NOT NULL,
  timestamp TIMESTAMP DEFAULT NOW()
);
```

**Used In**: costing-agent, auth-agent

---

## Caching Patterns

### Pattern 13: Local Token Validation

**Problem**: Validating every request against service is slow

**Solution**: Cache public keys, validate JWTs locally

**Implementation**:
```typescript
import jwt from 'jsonwebtoken';

class AuthAgent {
  private publicKeyCache: { key: string; expiresAt: number } | null = null;

  async validateToken(token: string): Promise<User> {
    // 1. Get public key (cached)
    const publicKey = await this.getPublicKey();

    // 2. Verify JWT locally (no network call!)
    try {
      const decoded = jwt.verify(token, publicKey, {
        algorithms: ['RS256']
      }) as JWTPayload;

      // 3. Check if token is blacklisted (quick Redis check)
      const isBlacklisted = await this.isTokenBlacklisted(decoded.jti);
      if (isBlacklisted) {
        throw new Error('Token revoked');
      }

      return decoded.user;
    } catch (error) {
      throw new Error('Invalid token');
    }
  }

  private async getPublicKey(): Promise<string> {
    // Check cache
    if (this.publicKeyCache && Date.now() < this.publicKeyCache.expiresAt) {
      return this.publicKeyCache.key;
    }

    // Fetch from service
    const response = await this.client.get('/jwks');
    const publicKey = response.data.keys[0].value;

    // Cache for 5 minutes
    this.publicKeyCache = {
      key: publicKey,
      expiresAt: Date.now() + 5 * 60 * 1000
    };

    return publicKey;
  }
}
```

**Performance**: ~0.5ms vs ~50ms for remote validation

**Used In**: auth-agent

---

## Intelligence Patterns

### Pattern 14: Behavioral Profile

**Problem**: Need to detect anomalies from normal behavior

**Solution**: Build 90-day behavioral profile per user

**Implementation**:
```typescript
interface BehaviorProfile {
  userId: string;
  dataPoints: number;
  typicalHours: number[];        // [9, 10, 11, ..., 17]
  typicalLocations: Location[];  // Cluster centroids
  typicalDevices: string[];      // Device fingerprints
  lastUpdated: Date;
}

class BehaviorAnalyzer {
  async buildProfile(userId: string): Promise<BehaviorProfile> {
    const history = await this.db.getLoginHistory(userId, { days: 90 });

    if (history.length < 10) {
      return null; // Insufficient data
    }

    return {
      userId,
      dataPoints: history.length,
      typicalHours: this.extractTypicalHours(history),
      typicalLocations: this.clusterLocations(history),
      typicalDevices: this.extractDevices(history),
      lastUpdated: new Date()
    };
  }

  detectAnomaly(current: LoginAttempt, profile: BehaviorProfile): number {
    let anomalyScore = 0;

    // Time anomaly
    if (!profile.typicalHours.includes(current.hour)) {
      anomalyScore += 30;
    }

    // Location anomaly
    if (!this.isNearTypicalLocation(current.location, profile.typicalLocations)) {
      anomalyScore += 50;
    }

    // Device anomaly
    if (!profile.typicalDevices.includes(current.deviceFingerprint)) {
      anomalyScore += 40;
    }

    return anomalyScore; // 0-100+
  }
}
```

**Used In**: auth-agent

---

### Pattern 15: Risk Scoring

**Problem**: Need continuous risk assessment, not binary

**Solution**: Weighted factor scoring (0-100)

**Implementation**:
```typescript
interface RiskFactors {
  device: number;      // 0-100
  location: number;
  velocity: number;
  behavior: number;
  credentials: number;
  recency: number;
}

const WEIGHTS = {
  device: 0.25,
  location: 0.20,
  velocity: 0.15,
  behavior: 0.25,
  credentials: 0.10,
  recency: 0.05
};

function calculateRiskScore(factors: RiskFactors): number {
  return Object.entries(factors).reduce(
    (sum, [key, value]) => sum + value * WEIGHTS[key],
    0
  );
}

// Usage
const factors = {
  device: scoreDevice(attempt),      // 0 = unknown, 100 = trusted
  location: scoreLocation(attempt),  // 0 = suspicious, 100 = typical
  velocity: scoreVelocity(attempt),  // 0 = impossible, 100 = normal
  behavior: scoreBehavior(attempt),  // 0 = anomalous, 100 = typical
  credentials: 80,                   // Password strength
  recency: 90                        // Recently active
};

const riskScore = calculateRiskScore(factors); // 0-100

if (riskScore >= 85) {
  return 'allow';
} else if (riskScore >= 60) {
  return 'require_mfa';
} else {
  return 'block';
}
```

**Used In**: auth-agent

---

## When to Use Each Pattern

| Pattern | Observer Agents | Controller Agents | Intelligence Level |
|---------|----------------|-------------------|-------------------|
| Auto-Registration | ✅ Required | ✅ Required | Any |
| Graceful Shutdown | ✅ Required | ✅ Required | Any |
| Heartbeat | ✅ Recommended | Optional | Any |
| Event Buffer | ✅ Required | ❌ N/A | 0-2 |
| Resilient Transmitter | ✅ Required | ✅ Required | Any |
| Compression | ✅ If high volume | Optional | Any |
| Graceful Degradation | ✅ Required | ⚠️ Design decision | Any |
| Health Checks | ✅ Required | ✅ Required | Any |
| Auto-Instrumentation | ✅ Recommended | ❌ N/A | 0-1 |
| Middleware | ❌ N/A | ✅ Required | Any |
| Local Validation | ❌ N/A | ✅ Required | Any |
| Hierarchical Tenancy | ✅ Required | ✅ Required | Any |
| Behavioral Profile | ❌ N/A | Optional | 3-4 |
| Risk Scoring | ❌ N/A | Optional | 3-4 |

---

## Pattern Combinations

### For Level 2 Observer Agent (like costing-agent):
1. Auto-Registration
2. Graceful Shutdown
3. Heartbeat
4. Event Buffer
5. Resilient Transmitter
6. Compression
7. Auto-Instrumentation
8. Hierarchical Tenancy

### For Level 4 Controller Agent (like auth-agent):
1. Auto-Registration
2. Graceful Shutdown
3. Middleware/Dependencies
4. Resilient Transmitter
5. Local Validation (caching)
6. Hierarchical Tenancy
7. Behavioral Profile
8. Risk Scoring

---

## Next Steps

- See [INTELLIGENCE_LAYERS.md](INTELLIGENCE_LAYERS.md) for intelligence patterns
- See [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md) for design methodology
- Check `components/` for reference implementations
