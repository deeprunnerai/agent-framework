# Active Controller Agent Template

Template for building **Level 2-4 Controller Agents** that make decisions and enforce policies, blocking application flow when needed.

## Use This Template When

- ✅ Agent makes decisions that affect application
- ✅ Need to block/allow requests
- ✅ Synchronous operation required
- ✅ Policy enforcement needed
- ✅ Optional: Learning and adaptation

## Examples

- **Auth Agent**: Intelligent authentication & authorization
- **Rate Limit Agent**: Enforce rate limits
- **Validation Agent**: Request validation
- **Compliance Agent**: Policy enforcement
- **Fraud Detection Agent**: Block fraudulent activity

## Architecture

```
Your Application
  ↓ (middleware blocks here)
SDK Agent (local validation)
  ↓ (if cache miss or decision needed)
Central Service (decision engine)
  ↓
Intelligence Layer (optional)
  ↓
Database
```

## Key Characteristics

- **Blocking**: Blocks application flow until decision made
- **Synchronous**: Request/response pattern
- **Cached**: Local validation for performance
- **Intelligent**: Optional learning and adaptation
- **Explainable**: All decisions have audit trail

## Components Included

### SDK Components
- Configuration Manager
- Lifecycle Manager
- Middleware (Express/FastAPI patterns)
- Local Validation Cache (JWT, policies)
- Request/Response Client
- Error Handling (fail-open vs fail-closed)

### Service Components
- Registration endpoint
- Validation/decision endpoint
- Policy management
- Authentication middleware
- Multi-tenancy (Org → Project → User)
- Audit logging (immutable)
- Health checks

### Intelligence Components (Optional - Level 3-4)
- Risk Scoring Engine
- Behavioral Analysis
- Anomaly Detection
- Decision Engine
- Learning Pipeline

## Integration Example

### Express.js (Node.js)
```typescript
import { AuthAgent } from '@myorg/auth-agent';

const auth = new AuthAgent({
  apiKey: process.env.AUTH_API_KEY,
  projectId: 'my-project'
});

// Protect routes
app.get('/api/protected',
  auth.middleware.requireAuth(),
  (req, res) => {
    // req.user populated by middleware
    res.json({ user: req.user });
  }
);

// Optional auth
app.get('/api/public',
  auth.middleware.optionalAuth(),
  (req, res) => {
    if (req.user) {
      return res.json({ personalized: true, user: req.user });
    }
    res.json({ personalized: false });
  }
);
```

### FastAPI (Python)
```python
from myorg_agent import AuthAgent, get_current_user

auth = AuthAgent(api_key=os.getenv("AUTH_API_KEY"))

@app.get("/api/protected")
async def protected(user: User = Depends(get_current_user)):
    return {"user": user}
```

## Performance Targets

**Critical** - Agent is in hot path!

- **Validation latency**: p95 < 50ms
- **Decision latency**: p95 < 200ms
- **Cache hit rate**: > 95%
- **Throughput**: 10,000 req/sec per instance
- **Availability**: 99.9%+

## Intelligence Levels

### Level 2: Rule-Based (Simple)
- Static policies
- Threshold-based decisions
- No learning
- Example: Rate limiting by IP

### Level 3: Pattern-Based (Adaptive)
- Learns behavioral patterns
- Dynamic thresholds
- Anomaly detection
- Example: Adaptive rate limiting

### Level 4: Autonomous (Intelligent)
- Risk scoring
- Behavioral learning
- Autonomous actions
- Self-improvement
- Example: Auth agent with threat detection

## Security Considerations

**Critical** - This agent enforces security!

- ✅ Authentication required for all endpoints
- ✅ Audit logging (immutable, all decisions)
- ✅ Secrets management (never log tokens)
- ✅ Fail-closed by default (deny on error)
- ✅ Rate limiting on validation endpoint
- ✅ Input validation (prevent injection)

## Fail-Open vs Fail-Closed

**Fail-Closed** (Default, Recommended):
```typescript
// On agent service error → Deny access
if (error) {
  throw new UnauthorizedError();
}
```

**Fail-Open** (Use with caution):
```typescript
// On agent service error → Allow access
if (error) {
  console.error('Agent failed, allowing access');
  return null; // Allow
}
```

**Recommendation**: Fail-closed for security-critical agents (auth, compliance), fail-open for non-critical (analytics).

## Caching Strategy

**Local Validation** (No network call):
```typescript
// Cache public keys for JWT validation
const publicKey = await this.getPublicKey(); // Cached 5 min
const decoded = jwt.verify(token, publicKey); // Local!
```

**Performance Improvement**: 0.5ms vs 50ms for remote validation

## Decision Explainability

Every decision must be explainable:

```typescript
interface Decision {
  decision: 'allow' | 'challenge' | 'block';
  reasoning: string;
  factors: Array<{
    name: string;
    value: number;
    weight: number;
    impact: 'positive' | 'negative';
  }>;
  alternatives: string[]; // What would change decision
  auditTrail: string;     // Unique ID for investigation
}
```

## Next Steps

1. Copy this template to your new project
2. Define your decision logic
3. Implement validation/decision endpoint
4. Add intelligence components (if Level 3-4)
5. Implement comprehensive testing
6. Security audit before production

## Reference Implementation

See `auth-agent` project for full production example.
