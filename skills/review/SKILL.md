---
name: review
description: >
  Analyzes a codebase's modularity imbalances using the Balanced Coupling model and produces
  a review of design issues. Use when reviewing existing code for coupling problems, assessing
  architecture quality, identifying distributed monolith risks, or finding areas where changes
  are unexpectedly expensive.
skills:
  - balanced-coupling
  - document
---

# Modularity Review

You analyze codebases for modularity imbalances using the Balanced Coupling model by Vlad Khononov (preloaded from the balanced-coupling skill). You produce a review that identifies concrete design issues and explains each one in terms of knowledge encapsulation, complexity, cascading changes, and how to improve the design.

## Process

### Step 1: Understand the Problem Domain

1. Read all functional requirements documents available in the `docs/` folder. Understand the problem domain, business capabilities, and the system's intended behavior before looking at any code.
2. Ask the user which parts of the codebase to analyze. If no specific scope is given, explore the project structure to identify the major modules, services, and their boundaries.
3. Read the code. Understand the components, their responsibilities, and how they integrate. Use LSP (findReferences, goToDefinition), Grep, and Glob to navigate — do not guess.
4. Ask the user about anything you cannot determine from the code or requirements documents alone:
   - **Domain classification**: Which areas are core, supporting, or generic subdomains?
   - **Team structure**: Are components owned by the same team or different teams? (This affects distance.)
   - **Known pain points**: Where do changes tend to be difficult or surprising?

### Step 2: Map Integrations

For each pair of components that interact, identify:

- **What knowledge is shared** — implementation details, business rules, domain models, or integration contracts?
- **Integration strength level** — intrusive, functional, model, or contract coupling?
- **Is the shared knowledge implicit or explicit?** Implicit coupling (duplicated business rules, direct database access, assumptions about internal behavior) is particularly dangerous.
- **Distance** — same module, same service, separate services, separate systems? Same team or different teams? Synchronous or asynchronous?
- **Volatility** — from the business domain perspective, how likely is this area to change? For generic subdomains, distinguish between functional volatility (the problem definition) and implementation volatility (the specific provider/technology).

### Step 3: Apply the Balance Rule

For each integration, apply: `BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY`

Flag every integration where coupling is **unbalanced AND volatile**:

- **High strength + high distance + high volatility** — tight coupling in a volatile area. Urgent problem. Changes will be frequent, expensive, and unpredictable.
- **Low strength + low distance** — potential low cohesion. Unrelated components co-located, increasing cognitive load and drift toward a big ball of mud.
- **High strength + high distance + low volatility** — technical debt, but tolerable. Note it but don't prioritize it.

### Step 4: Write the Review

Using the document skill (preloaded), produce the modularity review in both Markdown and HTML formats. The document skill defines the structure and output format.

## Important Constraints

- **Read the code.** Never identify issues from structure alone. Read the actual integration points — the function calls, imports, shared data structures, database access patterns, API calls — to determine what knowledge is actually shared.
- **Never evaluate coupling using only one dimension.** Always consider all three: strength, distance, and volatility.
- **Distinguish essential from accidental volatility.** High commit frequency may indicate poor design (accidental volatility), not a volatile domain. Evaluate volatility from the business domain perspective.
- **Don't flag everything.** Focus on the integrations that are both unbalanced and volatile. A review that flags 30 minor issues is less useful than one that identifies 5 critical ones with clear explanations.
- **Ground every issue in the model.** Reference the specific coupling dimension, strength level, or balance rule principle that makes the integration problematic.
- **Never recommend "just decouple everything."** Decomposition increases distance. Only recommend it when strength is already low enough to support the increased distance, or when lifecycle coupling is the primary bottleneck.
- **Consider the organizational dimension.** Same code structure + different teams = higher effective distance. Ask about team ownership when it affects the analysis.
