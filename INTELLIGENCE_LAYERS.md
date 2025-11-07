# Intelligence Layers

A taxonomy of intelligence levels for agents, from simple instrumentation to autonomous decision-making.

## Intelligence Hierarchy

```
Level 4: Autonomous Decision-Making
         ↑ (learns, decides, acts)
Level 3: Behavioral Learning
         ↑ (detects patterns, adapts)
Level 2: Pattern Detection
         ↑ (identifies anomalies)
Level 1: Aggregation & Analysis
         ↑ (summarizes data)
Level 0: Pure Instrumentation
         (collects raw data)
```

---

## Level 0: Pure Instrumentation

### Characteristics
- **Collects** raw data only
- **No processing** beyond formatting
- **No decisions** made
- **No learning** from data

### Capabilities
- Data capture (events, metrics, logs)
- Basic formatting
- Transmission to storage
- Timestamp and metadata attachment

### Example: Simple Logging Agent

```typescript
class LoggingAgent {
  track(event: LogEvent): void {
    const formattedEvent = {
      ...event,
      timestamp: new Date().toISOString(),
      hostname: os.hostname()
    };

    this.buffer.add(formattedEvent);
  }
}
```

### Use Cases
- Application logging
- Raw metrics collection
- Event streaming
- Basic telemetry

### Complexity: ⭐ (Very Low)

---

## Level 1: Aggregation & Analysis

### Characteristics
- **Aggregates** data (sum, count, average)
- **Basic statistics** (min, max, percentiles)
- **Simple calculations** (cost from tokens)
- **No learning** or prediction

### Capabilities
- Statistical aggregation
- Time-series bucketing
- Cost calculation from rates
- Basic reporting

### Example: Costing Agent

```typescript
class CostingAgent {
  // Collect raw events
  trackLLMCall(call: LLMCall): void {
    this.buffer.add({
      model: call.model,
      tokens: call.usage.total_tokens,
      timestamp: Date.now()
    });
  }

  // Server-side: Calculate costs
  async calculateCost(event: Event): Promise<number> {
    const pricing = await this.getPricing(event.model, event.timestamp);

    return (
      event.usage.input_tokens * pricing.input_per_token +
      event.usage.output_tokens * pricing.output_per_token
    );
  }

  // Aggregate by project/time
  async getProjectCosts(projectId: string, timeframe: string): Promise<Report> {
    return this.db.query(`
      SELECT
        DATE_TRUNC('day', timestamp) as day,
        model,
        SUM(tokens) as total_tokens,
        SUM(cost) as total_cost
      FROM events
      WHERE project_id = $1
        AND timestamp > NOW() - INTERVAL '${timeframe}'
      GROUP BY day, model
      ORDER BY day DESC
    `);
  }
}
```

### Use Cases
- Cost tracking (costing-agent)
- Performance metrics aggregation
- Usage analytics
- Billing calculations

### Complexity: ⭐⭐ (Low)

---

## Level 2: Pattern Detection

### Characteristics
- **Detects anomalies** from baselines
- **Identifies patterns** in data
- **Threshold-based alerting**
- **No learning** (static rules)

### Capabilities
- Anomaly detection (statistical)
- Threshold violation alerts
- Pattern matching
- Correlation analysis

### Example: Performance Monitoring Agent

```typescript
class PerformanceAgent {
  // Track metrics
  track(metric: Metric): void {
    this.buffer.add(metric);
  }

  // Detect anomalies (server-side)
  async detectAnomalies(metric: Metric): Promise<Anomaly[]> {
    const anomalies: Anomaly[] = [];

    // Get baseline (e.g., p95 over last 7 days)
    const baseline = await this.getBaseline(metric.name, { days: 7 });

    // Threshold check
    if (metric.value > baseline.p95 * 2) {
      anomalies.push({
        type: 'threshold_exceeded',
        severity: 'high',
        metric: metric.name,
        value: metric.value,
        threshold: baseline.p95 * 2
      });
    }

    // Sudden spike detection
    const recent = await this.getRecentValues(metric.name, { minutes: 5 });
    const avgRecent = average(recent);
    const avgBaseline = baseline.mean;

    if (avgRecent > avgBaseline * 3) {
      anomalies.push({
        type: 'sudden_spike',
        severity: 'medium',
        metric: metric.name,
        current: avgRecent,
        baseline: avgBaseline
      });
    }

    return anomalies;
  }
}
```

### Use Cases
- Performance anomaly detection
- Error rate alerting
- Capacity threshold monitoring
- SLA violation detection

### Complexity: ⭐⭐⭐ (Medium)

---

## Level 3: Behavioral Learning

### Characteristics
- **Learns** from historical data
- **Builds profiles** (user, system)
- **Adapts thresholds** dynamically
- **Predicts** based on patterns

### Capabilities
- Behavioral profiling
- Dynamic threshold adjustment
- Pattern learning
- Trend prediction
- Feedback loops

### Example: Auth Agent (Behavioral Analysis)

```typescript
class BehavioralAuthAgent {
  // Build user profile from 90-day history
  async buildProfile(userId: string): Promise<BehaviorProfile> {
    const history = await this.db.getLoginHistory(userId, { days: 90 });

    if (history.length < 10) {
      return null; // Insufficient data
    }

    return {
      userId,
      dataPoints: history.length,

      // Learn typical login times
      typicalHours: this.extractTimePatterns(history),
      // Hours where user logs in ≥10% of time

      // Learn typical locations (cluster analysis)
      typicalLocations: this.clusterLocations(history),
      // DBSCAN clustering of GPS coordinates

      // Learn typical devices
      typicalDevices: this.extractDeviceFingerprints(history),

      // Learn typical session duration
      avgSessionDuration: average(history.map(h => h.sessionDuration)),

      // Learn API usage patterns
      typicalAPIRate: this.calculateTypicalRate(userId),

      lastUpdated: new Date()
    };
  }

  // Detect behavioral anomaly
  async detectBehaviorAnomaly(
    attempt: LoginAttempt,
    profile: BehaviorProfile
  ): Promise<AnomalyScore> {
    let anomalyScore = 0;

    // Time anomaly
    if (!profile.typicalHours.includes(attempt.hour)) {
      anomalyScore += 30;
    }

    // Location anomaly
    const nearTypical = profile.typicalLocations.some(loc =>
      this.distance(attempt.location, loc) < 50 // km
    );
    if (!nearTypical) {
      anomalyScore += 50;
    }

    // Device anomaly
    if (!profile.typicalDevices.includes(attempt.deviceFingerprint)) {
      anomalyScore += 40;
    }

    return {
      score: anomalyScore,
      factors: {
        timeAnomaly: !profile.typicalHours.includes(attempt.hour),
        locationAnomaly: !nearTypical,
        deviceAnomaly: !profile.typicalDevices.includes(attempt.deviceFingerprint)
      }
    };
  }

  // Continuous learning: update profile after each login
  async updateProfile(userId: string, newLogin: Login): Promise<void> {
    const profile = await this.getProfile(userId);

    // Check if pattern significantly changed
    if (this.isSignificantChange(newLogin, profile)) {
      // Rebuild profile from scratch
      await this.buildProfile(userId);
    } else {
      // Incremental update
      await this.updateIncremental(profile, newLogin);
    }
  }
}
```

### Use Cases
- User behavior analysis (auth-agent)
- Fraud detection
- Predictive caching
- Adaptive rate limiting

### Complexity: ⭐⭐⭐⭐ (High)

---

## Level 4: Autonomous Decision-Making

### Characteristics
- **Makes decisions** independently
- **Takes actions** without approval
- **Explains reasoning** for decisions
- **Improves** from outcomes

### Capabilities
- Risk scoring and assessment
- Autonomous action execution
- Policy enforcement
- Self-tuning
- Explainable AI

### Example: Auth Agent (Full Intelligence)

```typescript
class IntelligentAuthAgent {
  // 1. Risk Scoring Engine
  async calculateRisk(attempt: LoginAttempt): Promise<RiskScore> {
    const factors = {
      device: await this.scoreDevice(attempt),          // 0-100
      location: await this.scoreLocation(attempt),      // 0-100
      velocity: await this.scoreVelocity(attempt),      // 0-100
      behavior: await this.scoreBehavior(attempt),      // 0-100
      credentials: this.scoreCredentials(attempt),      // 0-100
      recency: await this.scoreRecency(attempt.userId)  // 0-100
    };

    // Weighted aggregation
    const score = Object.entries(factors).reduce(
      (sum, [key, value]) => sum + value * this.WEIGHTS[key],
      0
    );

    return {
      score: Math.round(score),
      confidence: this.calculateConfidence(factors, attempt),
      factors,
      reasoning: this.explainScore(factors)
    };
  }

  // 2. Decision Engine
  async makeDecision(
    attempt: LoginAttempt,
    riskScore: RiskScore,
    anomalies: Anomaly[],
    context: Context
  ): Promise<AuthDecision> {
    // Critical anomalies = immediate block
    const critical = anomalies.filter(a => a.severity === 'critical');
    if (critical.length > 0) {
      return this.createBlockDecision(riskScore, critical);
    }

    // Context-aware decisions
    if (context.action === 'delete' || context.sensitiveData) {
      if (riskScore.score < 70) {
        return this.createMFADecision(riskScore);
      }
    }

    // Risk-based decisions
    if (riskScore.score >= 85) {
      return { decision: 'allow', reasoning: 'High trust score' };
    } else if (riskScore.score >= 60) {
      return { decision: 'challenge_mfa', reasoning: 'Medium trust, require MFA' };
    } else if (riskScore.score >= 30) {
      return { decision: 'admin_approval', reasoning: 'Low trust, require approval' };
    } else {
      return this.createBlockDecision(riskScore, anomalies);
    }
  }

  // 3. Autonomous Action Execution
  async executeDecision(decision: AuthDecision, attempt: LoginAttempt): Promise<void> {
    switch (decision.decision) {
      case 'allow':
        // Log successful auth
        await this.auditLog.log('auth_success', { attempt, decision });
        break;

      case 'challenge_mfa':
        // Send MFA challenge
        await this.mfa.sendChallenge(attempt.userId);
        await this.auditLog.log('mfa_required', { attempt, decision });
        break;

      case 'block':
        // Autonomous blocking
        await this.blockUser(attempt);
        await this.auditLog.log('auth_blocked', { attempt, decision });

        // Alert security team if critical
        if (decision.severity === 'critical') {
          await this.alerting.sendAlert('security', {
            type: 'critical_auth_block',
            user: attempt.userId,
            reason: decision.reasoning
          });
        }
        break;

      case 'admin_approval':
        // Escalate to human
        await this.approvals.create({
          type: 'auth_approval',
          user: attempt.userId,
          reason: decision.reasoning,
          expiresIn: '1 hour'
        });
        break;
    }
  }

  // 4. Autonomous Threat Response
  async respondToThreat(threat: ThreatDetection): Promise<void> {
    switch (threat.type) {
      case 'brute_force':
        // Block IP automatically
        await this.firewall.blockIP(threat.ip, { duration: '15m' });
        await this.auditLog.log('auto_block_ip', { threat });
        break;

      case 'credential_stuffing':
        // Revoke all user sessions
        await this.sessions.revokeAll(threat.userId);
        // Force password reset
        await this.users.forcePasswordReset(threat.userId);
        // Alert user
        await this.notifications.send(threat.userId, {
          type: 'security_alert',
          message: 'Suspicious activity detected. Password reset required.'
        });
        break;

      case 'impossible_travel':
        // Block new session, require email verification
        await this.sessions.block(threat.sessionId);
        await this.mfa.sendEmailVerification(threat.userId);
        break;

      case 'session_hijacking':
        // Kill session immediately
        await this.sessions.revoke(threat.sessionId);
        // Require re-auth
        await this.auditLog.log('session_hijack_detected', { threat });
        break;
    }
  }

  // 5. Learning from Outcomes
  async learnFromOutcome(decisionId: string, outcome: Outcome): Promise<void> {
    const decision = await this.db.getDecision(decisionId);

    if (outcome.wasFalsePositive) {
      // We blocked a legitimate user - adjust weights
      console.log('False positive detected, adjusting weights');

      for (const factor of decision.reasoning.factors) {
        if (factor.impact === 'negative') {
          // Decrease weight of factors that caused false positive
          await this.adjustWeight(factor.name, -0.02);
        }
      }

      // Record for analysis
      await this.metrics.increment('false_positives');

    } else if (outcome.wasFalseNegative) {
      // We allowed an attacker - tighten rules
      console.log('False negative detected, tightening rules');

      for (const factor of decision.reasoning.factors) {
        if (factor.score < 50) {
          // Increase weight of factors we should have caught
          await this.adjustWeight(factor.name, 0.05);
        }
      }

      await this.metrics.increment('false_negatives');
    }

    // Periodically retrain
    if (await this.shouldRetrain()) {
      await this.retrainWeights();
    }
  }

  // 6. Explainable Decisions
  explainDecision(decision: AuthDecision): Explanation {
    return {
      decision: decision.decision,
      primaryReason: decision.reasoning,
      factors: decision.riskScore.factors.map(f => ({
        name: f.name,
        score: f.score,
        weight: this.WEIGHTS[f.name],
        impact: f.impact,
        explanation: this.getFactorExplanation(f)
      })),
      alternatives: [
        `Would be ${decision.decision === 'allow' ? 'challenged' : 'allowed'} if trust score was ${decision.decision === 'allow' ? '<60' : '>85'}`,
        `Known device would increase score by ${20}`,
        `Typical location would increase score by ${15}`
      ],
      auditTrail: decision.auditId
    };
  }
}
```

### Use Cases
- Intelligent authentication (auth-agent)
- Autonomous security response
- Fraud prevention
- Adaptive systems

### Complexity: ⭐⭐⭐⭐⭐ (Very High)

---

## Intelligence Comparison Matrix

| Feature | L0 | L1 | L2 | L3 | L4 |
|---------|----|----|----|----|-----|
| **Data Collection** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Aggregation** | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Anomaly Detection** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Pattern Learning** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **Autonomous Decisions** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Self-Improvement** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Explainability** | N/A | N/A | ⚠️ | ⚠️ | ✅ |

---

## Choosing Intelligence Level

### Decision Framework

```
┌─────────────────────────────────────┐
│ Does agent need to make decisions?  │───No──► L0 or L1
└────────────┬────────────────────────┘
             │ Yes
      ┌──────▼───────┐
      │ Static rules │───Yes─► L2
      │  sufficient? │
      └──────┬───────┘
             │ No
      ┌──────▼───────┐
      │Need to learn │───Yes─► L3 or L4
      │  patterns?   │
      └──────┬───────┘
             │
      ┌──────▼───────┐
      │  Autonomous  │───Yes─► L4
      │   actions?   │
      └──────────────┘
```

### Cost vs Capability

| Level | Development Cost | Operational Cost | Maintenance | ROI |
|-------|-----------------|------------------|-------------|-----|
| L0 | ⭐ Low | ⭐ Low | ⭐ Low | ⭐⭐ |
| L1 | ⭐⭐ Medium | ⭐⭐ Medium | ⭐⭐ Medium | ⭐⭐⭐ |
| L2 | ⭐⭐⭐ Medium-High | ⭐⭐ Medium | ⭐⭐⭐ Medium | ⭐⭐⭐⭐ |
| L3 | ⭐⭐⭐⭐ High | ⭐⭐⭐ Medium-High | ⭐⭐⭐⭐ High | ⭐⭐⭐⭐ |
| L4 | ⭐⭐⭐⭐⭐ Very High | ⭐⭐⭐⭐ High | ⭐⭐⭐⭐⭐ Very High | ⭐⭐⭐⭐⭐ |

### Recommendation

- **Start simple**: Begin with L0 or L1
- **Add intelligence incrementally**: Don't jump to L4 immediately
- **Measure value**: Does added intelligence justify complexity?
- **Consider alternatives**: Sometimes simple rules (L2) are sufficient

---

## Implementation Patterns

### Layered Intelligence Architecture

```
┌─────────────────────────────────────────────┐
│         Application Layer                    │
│  (Uses agent via SDK/middleware)            │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         Agent SDK Layer                      │
│  (Local validation, caching, buffering)     │
└──────────────────┬──────────────────────────┘
                   │ HTTPS/REST
┌──────────────────▼──────────────────────────┐
│         Service API Layer                    │
│  (Authentication, rate limiting, routing)   │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│      Intelligence Layer (L2-L4)             │
│  ┌─────────────────────────────────────┐   │
│  │ L4: Decision Engine                  │   │
│  │      ↓                               │   │
│  │ L3: Behavioral Learning              │   │
│  │      ↓                               │   │
│  │ L2: Anomaly Detection                │   │
│  │      ↓                               │   │
│  │ L1: Aggregation                      │   │
│  │      ↓                               │   │
│  │ L0: Data Collection                  │   │
│  └─────────────────────────────────────┘   │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│         Data Layer                           │
│  (PostgreSQL, Redis, TimescaleDB)           │
└─────────────────────────────────────────────┘
```

### Incremental Intelligence

**Phase 1**: Build L0/L1 (MVP)
- Data collection
- Basic aggregation
- Reporting

**Phase 2**: Add L2 (Anomaly Detection)
- Statistical baselines
- Threshold alerts
- Pattern matching

**Phase 3**: Add L3 (Learning)
- Behavioral profiling
- Dynamic thresholds
- Trend prediction

**Phase 4**: Add L4 (Autonomous)
- Risk scoring
- Decision engine
- Self-tuning
- Explainability

---

## Next Steps

- Review [PATTERNS.md](PATTERNS.md) for intelligence implementation patterns
- See [AGENT_DESIGN_GUIDE.md](AGENT_DESIGN_GUIDE.md) for design methodology
- Check `components/intelligence/` for reference implementations
