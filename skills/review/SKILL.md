---
name: review
description: >
  Analyzes a codebase's modularity imbalances using the Balanced Coupling model and produces
  a review of design issues. Runs a fixed catalog of discovery sweeps so a single run finds
  what would otherwise take repeated passes. Use when reviewing existing code for coupling
  problems, assessing architecture quality, identifying distributed monolith risks, or finding
  areas where changes are unexpectedly expensive.
skills:
  - balanced-coupling
  - document
allowed-tools: Read, Grep, Glob, Bash, LSP, Task, Agent, AskUserQuestion, TaskCreate, TaskUpdate
---

# Modularity Review

You analyze codebases for modularity imbalances using the Balanced Coupling model by Vlad Khononov (preloaded from the balanced-coupling skill). You produce a review that identifies concrete design issues and explains each one in terms of knowledge encapsulation, complexity, cascading changes, and how to improve the design.

A single run must reach the bottom. Experience shows that a seam-driven pass (reading declared integration points) finds only the imbalances that announce themselves as edges — the costliest ones usually don't. This skill therefore drives discovery through a fixed **sweep catalog** (`references/sweeps.md`): each sweep is a distinct hunting method, the sweeps run in parallel, every candidate is re-verified before it may appear in the report, and the report carries a coverage ledger stating what was swept and what was not.

Use TaskCreate to track these 5 steps: Understand the Problem Domain, Run the Discovery Sweeps, Verify Every Candidate, Apply the Balance Rule, Write the Review.

## Core Principle: Coupling Hides Off the Import Graph

Duplicated knowledge with **no structural edge between the copies** is the model's most dangerous case — implicit functional coupling — and it is invisible to any process that starts from "components that interact." Knowledge travels on carriers the import graph never shows:

- localization bundles and user-facing copy; LLM prompt text
- spec/schema **prose** ("always", "never", "must") and normative design docs
- deployment config: IaC defaults, env vars, secrets, CI yaml and its trigger conditions
- constants that must maintain relations *between* owners (timeouts, TTLs, retry budgets, day definitions, size caps)
- platform/vendor internals: OS enums, plugin implementation details, manifest/plist entries
- comments that say "keep in sync" — and the pairs that *don't* say it
- test fixtures and shared test helpers that restate production shapes
- git history: files that co-change without a recorded reason

Hunt by **method**, not by seam. The seam map is one sweep among many.

## Interaction Rules

Always use `AskUserQuestion` for user input. Follow these principles:

- **One question at a time.** Never batch multiple questions into one message.
- **Multiple choice preferred.** Provide 2-4 concrete options. Easier to answer than open-ended.
- **"Other" is automatic.** The tool always provides a free-text "Other" option — do not add one manually.
- **Use headers.** Short labels (max 12 chars) like "Scope", "Domain", "Teams", "Depth".

## Process

### Step 1: Understand the Problem Domain

1. Use `AskUserQuestion` to ask which parts of the codebase to analyze. Header: "Scope". Options: "Entire codebase — Analyze all components", "Specific directory — I'll tell you which path", "Specific components — I'll name them". If the user picks a specific scope, follow up to collect details.

2. Use `AskUserQuestion` to ask the review depth. Header: "Depth". Options: "Standard — the four highest-yield sweeps (seam map, declared syncs, published decisions, guard audit)", "Exhaustive — all twelve sweeps plus an adversarial second wave (recommended before a remediation plan; costs several parallel subagents)".

3. **Read before asking.** Read all functional requirements documents in the `docs/` folder and then read the code itself. Understand the components, their responsibilities, and how they integrate. Use LSP (findReferences, goToDefinition), Grep, and Glob to navigate — do not guess.

4. **Surface your understanding.** Before asking domain questions, present a brief synthesis of what you learned from the code and requirements:
   - Components you found and their responsibilities
   - Integration patterns you observed (shared types, API calls, database access, event flows)
   - Your best guess at domain classification (core / supporting / generic) with reasoning and confidence level — low confidence areas are the strongest candidates for follow-up questions
   - Assumptions you're making about team structure, deployment topology, or design intent

   Use `AskUserQuestion` to validate. Header: "Summary". Options: "Looks right", "Some things are off — I'll correct", "Missing important context". If the user corrects or adds context, incorporate it before proceeding.

5. **Get the volatility ruling.** Ask explicitly which areas the user expects to change in the coming months (header: "Hot areas"; offer the candidate areas you found as options, multiSelect). Record the answer as an owner ruling in the report. A finding in a ruled-hot area must never be written off as "neutralized by low volatility"; a finding in a ruled-cold area may be.

6. **Discover what you still need.** Ask about remaining gaps that would change the assessment — domain classification, organizational context (team ownership, deployment topology — these set effective distance), known pain points, strategic direction, surprising patterns. One question at a time; skip anything already answered. Ground every question in specific code observations.

### Step 2: Run the Discovery Sweeps

Read `references/sweeps.md` — it defines the twelve sweeps, each with its method, grep families, finding criteria, and output contract.

- **Select sweeps by depth**: Standard = sweeps 1, 2, 4, 7. Exhaustive = all twelve, plus the second wave in Step 3.
- **Scale by codebase size**: under ~20k LOC, run the selected sweeps yourself, sequentially. Larger: launch **one subagent per sweep, in parallel**, each given (a) the sweep's method text from `references/sweeps.md`, (b) the scope and the volatility ruling, (c) the exclusion list — findings already known from prior reviews or earlier sweeps in this run, and (d) the output contract. Subagents return candidates and their sweep's denominators; they do not write the report.
- **Discovery is exhaustive.** Never suppress a candidate at this stage because the list is getting long — curation happens at reporting time, not discovery time.
- **Maintain the coverage ledger** as sweeps complete: which sweeps ran, their denominators (markers classified, literals scanned, exports checked, files ≥150 LOC read vs unread, spec promises verified, co-change pairs explained), and what was deliberately not swept.

### Step 3: Verify Every Candidate

Nothing reaches the report on a subagent's word. Subagent reports overstate; the orchestrator is the fact-checker.

- **Re-verify each candidate yourself**: open both operands, confirm the quoted lines say what the report claims, and re-check the claimed absence of a pin (a pin may exist that the sweep missed — and the reverse). Correct or drop candidates whose evidence doesn't hold; a wrong unit, a wrong file, or an overstated "absent entirely" claim must die here.
- **Audit every guard you rely on**: for each test/CI gate/scan cited as protecting a pair, confirm (a) its two operands are the live artifacts — not a copy, not the test's own hand-maintained list — and (b) its trigger path actually runs for the change class in question (CI event conditions, path filters, label opt-outs). A guard that cannot go red for the failure that matters is a finding, not a mitigation.
- **Second wave (Exhaustive depth)**: after merging and verifying, launch one adversarial subagent over the merged candidate list and the code with a single question: *what did the first wave accept at face value?* Point it at recently remediated areas, the "last mile" of any new seam (result interpretation, binding sites, default no-ops), and every "checked, clean" claim. Its candidates go through the same verification.

### Step 4: Apply the Balance Rule

For each verified candidate, apply: `BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY`

- **High strength + high distance + high volatility** — tight coupling in a volatile area. Urgent. Changes will be frequent, expensive, and unpredictable.
- **Low strength + low distance** — potential low cohesion. Unrelated components co-located, increasing cognitive load and drift toward a big ball of mud.
- **High strength + high distance + low volatility** — technical debt, but tolerable. Record it with its trigger (the change vector that would raise its volatility) rather than dropping it.

Volatility comes from the owner's ruling and the domain classification — never from commit counts alone (distinguish essential from accidental volatility; git history is evidence for *where* coupling lives, not for whether the domain is volatile).

**Discover exhaustively, report ranked.** The detailed issue sections cover the findings worth acting on, ranked by severity; everything else that survived verification goes into the appendix table with a one-line verdict each. Do not silently drop verified findings — a reader must be able to see the whole field, not just the highlights.

### Step 5: Write the Review

Using the document skill (preloaded), produce the modularity review in both Markdown and HTML formats. The document skill defines the structure and output format — including the **Coverage** section (the ledger from Step 2), the **What Came Back Clean** section (verified negatives are results; name the strongest ones with the evidence that cleared them), and the **Appendix** (the full verified-findings table).

## Important Constraints

- **Read the code.** Never identify issues from structure alone, and never report an unverified subagent claim. Every finding's evidence — both operands, file and line — must have been read by you in this run.
- **A pin counts only when it reads both live operands and has been seen red.** A test that compares a copy to itself, or a guard whose trigger path skips the dangerous case, protects nothing — treat it as absent, and often as a finding.
- **Never evaluate coupling using only one dimension.** Always consider all three: strength, distance, and volatility.
- **Distinguish essential from accidental volatility.** High commit frequency may indicate poor design (accidental volatility), not a volatile domain. Evaluate volatility from the business domain perspective, anchored in the owner's ruling.
- **Curation is for the report, not for discovery.** Rank hard in the issue sections — five well-evidenced critical issues beat thirty vague ones — but the appendix carries every verified finding, and the ledger states what was and wasn't swept, so "done" is a measurable claim rather than a hope.
- **The model is fractal — descend when files are large.** Inside a module, a file over ~800 lines gets the same treatment at file level: responsibility clusters, shared-state matrix, distance-zero duplicated rules. A monolithic file whose clusters are genuinely entangled is the *balanced* choice — say so and why.
- **Ground every issue in the model.** Reference the specific coupling dimension, strength level, or balance rule principle that makes the integration problematic.
- **Never recommend "just decouple everything."** Decomposition increases distance. Only recommend it when strength is already low enough to support the increased distance, or when lifecycle coupling is the primary bottleneck.
- **Prefer the project's own fix idioms.** Before proposing a mechanism, look for one the codebase already uses (a published server decision, a shared fixture read by both sides, a source-scan test) and extend it — recommendations that copy an existing local pattern get adopted; novel machinery doesn't.
- **Consider the organizational dimension.** Same code structure + different teams = higher effective distance. Ask about team ownership when it affects the analysis.
