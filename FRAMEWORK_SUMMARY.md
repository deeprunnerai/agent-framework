# Agent Framework - Visual Summary

A one-page reference for identifying, building, and operating effective agents.

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║                   🤖  AGENT FRAMEWORK - CORE PRINCIPLES                      ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝


┌──────────────────────────────────────────────────────────────────────────────┐
│  1️⃣  DON'T BUILD AGENTS FOR EVERYTHING                                       │
└──────────────────────────────────────────────────────────────────────────────┘

Use agents ONLY for: AMBIGUOUS + HIGH-VALUE + VERIFIABLE tasks

                   Can you map the decision tree?
                              ↓
                    ┌─────────┴─────────┐
                   YES                 NO
                    │                   │
              Use WORKFLOW        Consider AGENT
              • Known steps       • Exploratory
              • Predictable       • Uncertain iterations
              • Deterministic     • Context-dependent

Evaluate with 3 factors:

  ① COMPLEXITY: Is it exploratory?          ② VALUE: Token cost < benefit?
     └─ If flowchart works → Workflow          └─ Break-even < 1 month?
     └─ If uncertain path → Agent              └─ Recurring problem?

  ③ ERROR COST: Can mistakes be caught?
     └─ LOW      → Safe for agents
     └─ MEDIUM   → Agents + validation
     └─ HIGH     → Agents + human approval
     └─ CRITICAL → Never use agents


┌──────────────────────────────────────────────────────────────────────────────┐
│  2️⃣  KEEP IT SIMPLE                                                           │
└──────────────────────────────────────────────────────────────────────────────┘

           ┌───────────────────────────────────────┐
           │       EVERY AGENT = 3 PARTS           │
           ├───────────────────────────────────────┤
           │                                       │
           │   ┌───────┐   ┌──────┐   ┌─────────┐│
           │   │ MODEL │ → │TOOLS │ → │   ENV   ││
           │   │       │   │      │   │         ││
           │   │Thinks │   │ Acts │   │Observes ││
           │   │Decides│   │ Calls│   │Feedback ││
           │   └───────┘   └──────┘   └─────────┘│
           │        ↑          ↑           │      │
           │        └──────────┴───────────┘      │
           │           (Feedback Loop)             │
           └───────────────────────────────────────┘

Start with SIMPLEST version:
  • Model: Rule-based (if X then Y)
  • Tools: 2-3 essential actions
  • Environment: Minimal state

Iterate on:
  • Prompt (goal + behavior)
  • Tools (what it can do)
  • Feedback (what it sees)

Complexity later. Clarity first.


┌──────────────────────────────────────────────────────────────────────────────┐
│  3️⃣  THINK LIKE YOUR AGENT                                                    │
└──────────────────────────────────────────────────────────────────────────────┘

           YOUR VIEW              vs.          AGENT VIEW
    ┌─────────────────────┐          ┌─────────────────────┐
    │ • Complete codebase │          │ • Context window    │
    │ • Git history       │          │ • Tools available   │
    │ • Documentation     │          │ • Environment state │
    │ • Team knowledge    │          │ • No memory beyond  │
    │ • Business context  │          │ • No intuition      │
    │ • Common sense      │          │ • No common sense   │
    └─────────────────────┘          └─────────────────────┘

Debug from agent's perspective:
  ❓ What information did it have?
  ❓ What tools could it use?
  ❓ Was the goal measurably defined?
  ❓ Did tools return clear signals?
  ❓ Did environment update after actions?

Agent only knows what you give it: Model + Tools + Environment


┌──────────────────────────────────────────────────────────────────────────────┐
│  4️⃣  MAKE IT BUDGET & ERROR-AWARE                                             │
└──────────────────────────────────────────────────────────────────────────────┘

CONTROL: Tokens • Latency • Error Impact

Add EARLY:
  ✓ Feedback loops (how does agent know it succeeded?)
  ✓ Caching (local validation, avoid network calls)
  ✓ Evaluation harnesses (automated test scenarios)

            ┌─────────────────────────────────────┐
            │    EVALUATION HARNESS                │
            ├─────────────────────────────────────┤
            │ Test Scenario → Run Agent → Verify  │
            │                                      │
            │ Week 1: Build + 3 scenarios          │
            │ Week 2: Add 5 more scenarios         │
            │ Week 3: Run in CI pipeline           │
            │ Week 4+: Add scenario per bug        │
            └─────────────────────────────────────┘

Target metrics:
  • Pass rate: > 95%
  • Avg iterations: < 5
  • Convergence rate: > 95%
  • Cost per goal: Track & optimize


┌──────────────────────────────────────────────────────────────────────────────┐
│  5️⃣  EVOLVE INTENTIONALLY                                                     │
└──────────────────────────────────────────────────────────────────────────────┘

SINGLE → MULTI-AGENT: Only when needed!

Add second agent when:
  • Goals fundamentally conflict
  • Domains completely distinct
  • Operational rhythms differ
  • Failure isolation critical

DON'T add second agent for:
  • "Might be useful later"
  • Can be solved with better tools
  • Separation of concerns within one agent

     ╔════════════════════════════════════════╗
     ║      INCREMENTAL DEVELOPMENT PATH       ║
     ╠════════════════════════════════════════╣
     ║                                         ║
     ║  Week 1-2: Level 2 (Rule-Based)        ║
     ║    if X then Y                          ║
     ║    → Ship quickly, validate             ║
     ║                                         ║
     ║  Week 3-4: Level 3 (Pattern Detection) ║
     ║    Baseline + anomaly detection         ║
     ║    → A/B test, reduce false positives  ║
     ║                                         ║
     ║  Week 5-8: Level 4 (Autonomous)        ║
     ║    Risk scoring, multi-factor           ║
     ║    → Gradual rollout with oversight    ║
     ║                                         ║
     ║  Month 3+: Optimize & Refine           ║
     ║    Improve loop efficiency, reduce cost ║
     ║    → Add LLM enhancement if needed     ║
     ║                                         ║
     ╚════════════════════════════════════════╝


┌──────────────────────────────────────────────────────────────────────────────┐
│  THE CORE AGENT LOOP                                                         │
└──────────────────────────────────────────────────────────────────────────────┘

Every agent operates on this cycle:

        ┌─────────┐
        │  GOAL   │  Define success criteria (measurable!)
        └────┬────┘
             ↓
        ┌────────┐
        │  PLAN  │  Decide next action based on state
        └────┬───┘
             ↓
        ┌────────┐
        │  ACT   │  Execute tools/actions
        └────┬───┘
             ↓
        ┌─────────┐
        │ OBSERVE │  Collect feedback/results
        └────┬────┘
             ↓
        ┌─────────┐
        │ REFLECT │  Did it work? Adjust strategy?
        └────┬────┘
             ↓
      ┌──────┴───────┐
      │ REPEAT or    │  Goal achieved or max iterations?
      │ EXIT         │
      └──────────────┘

CRITICAL:
  • maxIterations: Prevent infinite loops (target: < 5)
  • successCriteria: Measurable goal verification
  • Tool feedback: Clear success/failure signals
  • State refresh: Environment updates after actions


╔══════════════════════════════════════════════════════════════════════════════╗
║                              DECISION MATRIX                                  ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  Complexity │  Value  │ Error Cost │ Recommendation                          ║
║  ───────────┼─────────┼────────────┼─────────────────────────────────────   ║
║  Low        │  Any    │ Any        │ ❌ WORKFLOW - Don't use agent          ║
║  High       │  Low    │ Any        │ ❌ MANUAL - Not worth automation       ║
║  High       │  High   │ Critical   │ ❌ HUMAN + TOOLS - Too risky           ║
║  High       │  High   │ Low-Medium │ ✅ AGENT - Good fit                    ║
║  High       │  High   │ High       │ ⚠️  AGENT + APPROVAL - Supervised      ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝


╔══════════════════════════════════════════════════════════════════════════════╗
║                              QUICK CHECKLIST                                  ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  Before Building:                                                            ║
║    □ Can workflow solve this? (deterministic steps?)                         ║
║    □ Is value > token cost + complexity cost?                                ║
║    □ Can mistakes be caught/reversed?                                        ║
║    □ Is feedback measurable?                                                 ║
║                                                                              ║
║  While Building:                                                             ║
║    □ Model: Started with simplest (rule-based)?                              ║
║    □ Tools: 2-3 essential actions, idempotent?                               ║
║    □ Environment: Minimal state that updates?                                ║
║    □ Goal: Clear success criteria defined?                                   ║
║    □ Loop: maxIterations < 5 target?                                         ║
║                                                                              ║
║  After Building:                                                             ║
║    □ Evaluation harness with 3+ scenarios?                                   ║
║    □ Pass rate > 95%?                                                        ║
║    □ Avg iterations < 5?                                                     ║
║    □ Cost per goal tracked?                                                  ║
║    □ Running in CI pipeline?                                                 ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝


═══════════════════════════════════════════════════════════════════════════════

                            💡 REMEMBER

  ✓ Identify WHEN agents make sense (not everything!)
  ✓ Build SIMPLE, looping systems (Model + Tools + Environment)
  ✓ Debug from THEIR point of view (limited context)
  ✓ Scale with BUDGETS and care (harnesses, metrics, monitoring)
  ✓ Evolve INTENTIONALLY (Level 2 → 3 → 4, single → multi)

═══════════════════════════════════════════════════════════════════════════════

                       📚 FULL DOCUMENTATION

  • README.md - Overview, taxonomy, when to build
  • CORE_AGENT_MECHANICS.md - Loop, tools, verification, metrics
  • PATTERNS.md - Core Agent Loop + architectural patterns
  • MULTI_AGENT_PATTERNS.md - Multi-agent coordination & safety
  • INTELLIGENCE_LAYERS.md - Levels 0-4 intelligence guide
  • AGENT_DESIGN_GUIDE.md - 6-phase design methodology
  • DECISION_TREE.md - Step-by-step architectural decisions

═══════════════════════════════════════════════════════════════════════════════
```
