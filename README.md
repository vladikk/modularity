# Modularity Skills

**TL;DR:** A [Claude Code](https://code.claude.com) plugin for designing and analyzing modular software systems using the [Balanced Coupling](https://coupling.dev) model.

There's no shortage of AI tools that provide code-level feedback: best practices, edge cases, potential bugs. That's useful, but it's not where the costly mistakes hide. In the AI era, code is generated faster than ever, and so technical debt accumulates faster too. Any architectural inefficiency, any misdrawn boundary, any unmanaged coupling will grow into a big ball of mud at a pace that wasn't possible before.

This plugin operates at the architectural level. It includes two skills:

- **`/modularity:review`** analyzes an existing codebase for coupling imbalances: what knowledge is properly encapsulated, what's leaking across component boundaries, and where cascading changes are waiting to happen.
- **`/modularity:high-level-design`** goes the other direction, designing modular architectures from functional requirements and producing module design docs with integration contracts, test specifications, and a full coupling assessment.

Both skills are grounded in the [Balanced Coupling](https://coupling.dev) model, so every recommendation traces back to a concrete dimension (integration strength, distance, volatility), not gut feel.

## Installation

Requires [Claude Code](https://code.claude.com/docs/en/quickstart) v1.0.33 or later.

Add the marketplace and install the plugin:

```
/plugin marketplace add vladikk/modularity
/plugin install modularity@vladikk-modularity
```

Alternatively, clone the repo and load it directly:

```bash
git clone https://github.com/vladikk/modularity.git
claude --plugin-dir ./modularity
```

## Skills

### `/modularity:review` — Modularity Review

Analyzes an existing codebase for coupling imbalances. Use when:

- Reviewing code for coupling problems
- Assessing architecture quality
- Identifying distributed monolith risks
- Investigating why changes in one area break things elsewhere

**How it works:**

1. Reads your code and functional requirements
2. Asks you about domain classification (core/supporting/generic), team structure, and known pain points
3. Maps integrations between components across three dimensions: integration strength, distance, and volatility
4. Applies the balance rule to flag imbalances
5. Produces a review document (Markdown + HTML) with a coupling overview table, issue descriptions, and concrete improvement recommendations

**Usage:**

```
/modularity:review
```

The review output includes hyperlinks to [coupling.dev](https://coupling.dev) for each coupling concept referenced.

### `/modularity:high-level-design` — High-Level Design

Designs modular architectures from functional requirements. Use when:

- Designing a new system from scratch
- Creating architecture documentation for an existing system
- Producing module-level design specs with integration contracts and test specifications

**How it works:**

1. Reads your functional requirements and asks clarifying questions
2. Classifies domain areas by volatility (core/supporting/generic subdomains)
3. Designs the modular architecture, assessing coupling across all three dimensions
4. Produces design documents for each module (responsibilities, encapsulated knowledge, integration contracts, change vectors)
5. Produces test specifications for each module
6. Produces an architecture overview document
7. Self-reviews the design for modularity imbalances and iterates until clean

Each step requires your approval before proceeding.

**Usage:**

```
/modularity:high-level-design
```

## The Balanced Coupling Model

Both skills are grounded in the Balanced Coupling model, which evaluates coupling across three dimensions:

- **Integration Strength** — how much knowledge is shared between components (intrusive > functional > model > contract)
- **Distance** — the socio-technical cost of co-evolving components (code structure, team boundaries, runtime dependencies)
- **Volatility** — the probability of a component needing to change, evaluated from the business domain perspective

The balance rule: `BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY`

Coupling is balanced when strength and distance counterbalance each other, or when volatility is low enough to neutralize the imbalance.

Learn more at [coupling.dev](https://coupling.dev).

## License

This work is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). For commercial use, contact skill at coupling dot dev.

## AI Training Restriction

This repository and its contents may not be used for training, fine-tuning, or any other form of machine learning model development without explicit written permission from the author.
