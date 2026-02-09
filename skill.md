---
name: llm-council
description: "Use when a task needs stress-testing from multiple cognitive angles, cross-validation, or when the user mentions council, parliamentary siege, swarm, or multi-agent review. Dispatches 5 specialized subagents to draft, attack, and synthesize a robust response."
---

# LLM Council — Parliamentary Siege

Dispatch 5 subagents with distinct cognitive roles to stress-test any task. One drafts, three attack, one synthesizes.

**Core principle:** A single perspective has blind spots. Five adversarial perspectives produce robust outputs.

## When to Use

- User says "council", "siege", "swarm", or "multi-agent"
- Task benefits from adversarial stress-testing (architecture decisions, security reviews, research, complex code)
- High-stakes output where mistakes are costly
- User wants cross-validated answers

**Don't use for:** Simple one-line fixes, file lookups, or tasks where speed matters more than depth.

## Verbosity

- **Default:** Hide internal debate, show only final ratified output
- **Verbose mode:** User says "verbose council" or "show debate" → show each agent's output

## The Five Roles

| Role | Agent | Focus | Default Model |
|------|-------|-------|---------------|
| **Alpha** | Generator | Depth, nuance, comprehensive first draft | claude-opus-4.6 |
| **Beta** | Red Teamer | Break logic, find vulnerabilities, counter-arguments | gpt-5.2 |
| **Gamma** | Fact-Checker | Grounding, accuracy, real-world validity | gemini-3-pro-preview |
| **Delta** | Optimizer | Efficiency, structure, clarity, UX | claude-sonnet-4 |
| **Epsilon** | Synthesizer | Merge all feedback into final ratified output | claude-opus-4.6 |

Models are recommended defaults — adapt based on task domain and availability.

## The Protocol

```
Phase 1: DRAFT
  Alpha generates comprehensive response to the task

Phase 2: SIEGE (parallel)
  Beta attacks logic, security, edge cases
  Gamma verifies claims, checks accuracy
  Delta critiques structure, efficiency, clarity

Phase 3: SYNTHESIZE
  Epsilon reads draft + all critiques
  If MAJOR disagreements found → Alpha revises, re-enter Phase 2
  If minor or no disagreements → produce final output

Phase 4: RATIFY
  Present final output to user
```

## Implementation

### Phase 1 — Draft (Alpha)

Dispatch a `general-purpose` subagent with model override:

```
task(
  agent_type: "general-purpose",
  model: "claude-opus-4.6",
  prompt: "You are Alpha, the Primary Generator on an LLM Council.
Your role: Create a comprehensive, nuanced response to this task.
Be thorough — your draft will be attacked by three adversarial reviewers.

TASK: {user_task}

Provide your complete draft response."
)
```

### Phase 2 — Siege (Beta, Gamma, Delta in parallel)

Dispatch three subagents **simultaneously**:

**Beta (Red Teamer):**
```
task(
  agent_type: "general-purpose",
  model: "gpt-5.2",
  prompt: "You are Beta, the Red Teamer on an LLM Council.
Your job: ATTACK this draft. Find every flaw.

Focus on:
- Logical fallacies or unsupported claims
- Security vulnerabilities or safety issues
- Edge cases not considered
- Counter-arguments the draft ignores
- Assumptions that could be wrong

Be adversarial. If you can't find real flaws, say so — don't invent problems.

DRAFT TO ATTACK:
{alpha_draft}

ORIGINAL TASK:
{user_task}

Output: List each flaw with severity (CRITICAL / IMPORTANT / MINOR) and explanation."
)
```

**Gamma (Fact-Checker):**
```
task(
  agent_type: "general-purpose",
  model: "gemini-3-pro-preview",
  prompt: "You are Gamma, the Fact-Checker on an LLM Council.
Your job: VERIFY every factual claim in this draft.

Focus on:
- Are technical claims accurate?
- Are recommendations grounded in real-world practice?
- Are there citations or evidence for key claims?
- Does anything contradict known best practices?
- Are version numbers, API references, or tool names correct?

Use web_search to verify claims when possible.

DRAFT TO VERIFY:
{alpha_draft}

ORIGINAL TASK:
{user_task}

Output: List each claim checked with verdict (VERIFIED / UNVERIFIED / INCORRECT) and evidence."
)
```

**Delta (Optimizer):**
```
task(
  agent_type: "general-purpose",
  model: "claude-sonnet-4",
  prompt: "You are Delta, the Optimizer on an LLM Council.
Your job: CRITIQUE the structure, efficiency, and usability of this draft.

Focus on:
- Is the response well-organized and scannable?
- Is there unnecessary verbosity or repetition?
- Could the format be improved (tables, lists, code blocks)?
- Is the response actionable and practical?
- Does it match the user's likely experience level?
- Is anything missing that the user would need?

DRAFT TO CRITIQUE:
{alpha_draft}

ORIGINAL TASK:
{user_task}

Output: List each issue with severity (CRITICAL / IMPORTANT / MINOR) and suggested improvement."
)
```

### Phase 3 — Synthesize (Epsilon)

After all three siege agents return:

```
task(
  agent_type: "general-purpose",
  model: "claude-opus-4.6",
  prompt: "You are Epsilon, the Synthesizer on an LLM Council.
You have the original draft and three adversarial reviews.

Your job:
1. Assess if there are MAJOR disagreements (any CRITICAL flaws found)
2. If MAJOR: Output 'VERDICT: REVISE' and list what must change
3. If no MAJOR issues: Produce the FINAL ratified response by:
   - Starting from Alpha's draft
   - Incorporating valid critique from Beta, Gamma, Delta
   - Removing anything flagged as incorrect by Gamma
   - Applying structural improvements from Delta
   - Addressing security/logic issues from Beta

ORIGINAL TASK:
{user_task}

ALPHA DRAFT:
{alpha_draft}

BETA (Red Team) FINDINGS:
{beta_output}

GAMMA (Fact-Check) FINDINGS:
{gamma_output}

DELTA (Optimizer) FINDINGS:
{delta_output}

Output either:
- 'VERDICT: REVISE' + revision instructions (if critical flaws)
- 'VERDICT: RATIFIED' + the final polished response (if no critical flaws)"
)
```

### Phase 3b — Revision Loop (if needed)

If Epsilon says REVISE:
1. Send revision instructions back to Alpha as a new subagent
2. Re-run Phase 2 (Siege) on the revised draft
3. Re-run Phase 3 (Synthesize)
4. Maximum 2 revision rounds — after that, Epsilon must ratify with caveats

### Phase 4 — Ratify

- **Default mode:** Present only the ratified response to the user
- **Verbose mode:** Show each phase's output with headers:
  ```
  ## 📝 Alpha Draft
  ...
  ## ⚔️ Beta (Red Team)
  ...
  ## ✅ Gamma (Fact-Check)
  ...
  ## 🔧 Delta (Optimizer)
  ...
  ## 🏛️ Ratified Verdict
  ...
  ```

## Domain Adaptation

Adjust agent focus based on task type:

| Domain | Beta Focus | Gamma Focus | Delta Focus |
|--------|-----------|-------------|-------------|
| **Code** | Security, edge cases, race conditions | API accuracy, version correctness | Performance, readability, patterns |
| **Architecture** | Failure modes, scalability limits | Technology claims, benchmarks | Diagram clarity, completeness |
| **Research** | Bias, methodology flaws | Source verification, citations | Readability, actionability |
| **Writing** | Logical consistency, tone | Factual accuracy | Flow, conciseness, formatting |

## Common Mistakes

- **Don't skip the siege** — The whole point is adversarial review
- **Don't run agents sequentially** — Beta/Gamma/Delta MUST run in parallel
- **Don't force disagreements** — If the draft is solid, say so
- **Don't exceed 2 revision rounds** — Diminishing returns; ratify with caveats
- **Don't use for trivial tasks** — This is expensive; use proportionally
