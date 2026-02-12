---
name: llm-council
description: "Use when a task needs stress-testing from multiple cognitive angles, cross-validation, or when the user mentions council, parliamentary siege, swarm, or multi-agent review. Dispatches 3 specialized subagents in parallel plus an orchestrator for fast, robust results."
---

# LLM Council — Fast Triad

Dispatch 3 subagents in parallel with distinct cognitive roles, then orchestrate a final synthesis. Optimized for speed without sacrificing rigor.

**Core principle:** Three diverse perspectives from different model families catch blind spots fast. An orchestrator merges them into a polished result.

## When to Use

- User says "council", "siege", "swarm", or "multi-agent"
- Task benefits from adversarial stress-testing (architecture decisions, security reviews, research, complex code)
- High-stakes output where mistakes are costly
- User wants cross-validated answers

**Don't use for:** Simple one-line fixes, file lookups, or tasks where speed matters more than depth.

## Verbosity

- **Default:** Hide internal debate, show only final ratified output
- **Verbose mode:** User says "verbose council" or "show debate" → show each agent's output

## The Four Roles

| Role | Agent | Focus | Default Model | Fallback |
|------|-------|-------|---------------|----------|
| **Alpha** | Drafter & Red Teamer | Comprehensive draft + self-critique of weaknesses | claude-opus-4.6 | gpt-5.2 |
| **Beta** | Fact-Checker & Validator | Accuracy, grounding, real-world validity, edge cases | gpt-5.2 | gemini-3-pro-preview |
| **Gamma** | Optimizer & Devil's Advocate | Structure, efficiency, counter-arguments, alternatives | gemini-3-pro-preview | claude-opus-4.6 |
| **Orchestrator** | Synthesizer | Merge all outputs into final ratified response | claude-opus-4.6 | gpt-5.2 |

Each subagent uses a **different model family** to maximize cognitive diversity. If a model is unavailable, fall back to the next in the chain.

## The Protocol

```
Phase 1: PARALLEL TRIAD (all 3 subagents run simultaneously)
  Alpha drafts comprehensive response + flags own uncertainties
  Beta fact-checks the task requirements, verifies assumptions, identifies edge cases
  Gamma proposes alternative approaches, critiques structure, optimizes for clarity

Phase 2: ORCHESTRATE
  Orchestrator reads all 3 outputs
  Merges the best elements, resolves conflicts, produces final output
  If CRITICAL conflicts exist → flag them with caveats (no revision loop for speed)

Phase 3: RATIFY
  Present final output to user
```

## Implementation

### Phase 1 — Parallel Triad (Alpha, Beta, Gamma simultaneously)

Dispatch all three subagents **at the same time** using parallel tool calls:

**Alpha (Drafter & Red Teamer):**
```
task(
  agent_type: "general-purpose",
  model: "claude-opus-4.6",
  prompt: "You are Alpha on an LLM Council (Fast Triad mode).
Your dual role: Create a comprehensive response AND red-team your own work.

TASK: {user_task}

Instructions:
1. Write a thorough, nuanced response to the task
2. Then add a section '## Self-Critique' where you:
   - Flag any assumptions you made
   - Identify weaknesses or edge cases in your response
   - Note areas where you're uncertain
   - List potential counter-arguments

Be thorough in both the draft AND the self-critique."
)
```

**Beta (Fact-Checker & Validator):**
```
task(
  agent_type: "general-purpose",
  model: "gpt-5.2",
  prompt: "You are Beta on an LLM Council (Fast Triad mode).
Your role: Independent fact-checking and validation of the task requirements.

TASK: {user_task}

Instructions:
1. Analyze the task independently — produce your OWN solution/response
2. Focus especially on:
   - Factual accuracy of any claims or technical details
   - Edge cases and boundary conditions
   - Security or safety considerations
   - Real-world validity and practicality
   - Version numbers, API correctness, tool names
3. Use web_search to verify claims when possible
4. Flag anything with severity: CRITICAL / IMPORTANT / MINOR

Output your independent response followed by a '## Validation Notes' section."
)
```

**Gamma (Optimizer & Devil's Advocate):**
```
task(
  agent_type: "general-purpose",
  model: "gemini-3-pro-preview",
  prompt: "You are Gamma on an LLM Council (Fast Triad mode).
Your role: Propose the most elegant, efficient solution AND play devil's advocate.

TASK: {user_task}

Instructions:
1. Produce your OWN response optimized for:
   - Clarity and scannability
   - Minimal complexity (simplest viable approach)
   - Actionability (user can execute immediately)
   - Proper formatting (tables, lists, code blocks as appropriate)
2. Then add a '## Devil's Advocate' section where you:
   - Argue against the obvious approach
   - Propose at least one alternative solution
   - Identify what could go wrong
   - Question unstated assumptions

Be concise but thorough."
)
```

### Phase 2 — Orchestrate

After all three subagents return, the **main agent** (you) acts as Orchestrator:

```
task(
  agent_type: "general-purpose",
  model: "claude-opus-4.6",
  prompt: "You are the Orchestrator on an LLM Council (Fast Triad mode).
You have three independent responses to the same task from different AI models.

Your job — produce a SINGLE final response by:
1. Identifying consensus across the three responses
2. Resolving conflicts by picking the best-supported position
3. Incorporating the strongest elements from each:
   - Alpha's depth and self-identified uncertainties
   - Beta's fact-checked details and validation
   - Gamma's structural clarity and alternative perspectives
4. If any CRITICAL conflicts remain unresolved, note them as caveats
5. Produce a clean, polished final output — not a meta-commentary

ORIGINAL TASK:
{user_task}

ALPHA (Drafter & Red Teamer) OUTPUT:
{alpha_output}

BETA (Fact-Checker & Validator) OUTPUT:
{beta_output}

GAMMA (Optimizer & Devil's Advocate) OUTPUT:
{gamma_output}

Output: The final ratified response. Do NOT include agent names or meta-discussion unless there are critical unresolved conflicts."
)
```

### Phase 3 — Ratify

- **Default mode:** Present only the orchestrated response to the user
- **Verbose mode:** Show each agent's output with headers:
  ```
  ## 📝 Alpha (Draft + Self-Critique)
  ...
  ## ✅ Beta (Fact-Check + Validation)
  ...
  ## 🔧 Gamma (Optimized + Devil's Advocate)
  ...
  ## 🏛️ Orchestrated Verdict
  ...
  ```

## Domain Adaptation

Adjust agent focus based on task type:

| Domain | Alpha Focus | Beta Focus | Gamma Focus |
|--------|------------|-----------|-------------|
| **Code** | Implementation + security self-review | API accuracy, version correctness, edge cases | Performance, readability, alternative patterns |
| **Architecture** | System design + failure mode analysis | Technology claims, benchmarks, scalability | Diagram clarity, simplicity, alternatives |
| **Research** | Comprehensive analysis + bias check | Source verification, citations, methodology | Readability, actionability, counter-arguments |
| **Writing** | Content + tone self-critique | Factual accuracy, consistency | Flow, conciseness, formatting |

## Key Differences from Full Council (5-agent)

| Aspect | Fast Triad (3+1) | Full Council (5) |
|--------|-------------------|-------------------|
| Subagents | 3 parallel | 1 sequential + 3 parallel + 1 sequential |
| Total phases | 2 (parallel + orchestrate) | 4 (draft → siege → synthesize → ratify) |
| Revision loops | None (caveats instead) | Up to 2 rounds |
| Speed | ~2x faster | More thorough |
| Model diversity | 3 different families | 2-3 families |

## Common Mistakes

- **Don't run agents sequentially** — All three MUST run in parallel for speed
- **Don't force disagreements** — If agents agree, that's a strong signal
- **Don't add revision loops** — Speed is the point; use caveats for unresolved issues
- **Don't use for trivial tasks** — Still has overhead; use proportionally
- **Don't skip the orchestrator** — Raw agent outputs need merging, not concatenation
