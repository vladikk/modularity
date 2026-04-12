# Balanced Coupling — Extended Reference

Supplementary detail for the Balanced Coupling model. Loaded on demand when deeper analysis requires connascence-level granularity or DDD pattern mapping beyond what SKILL.md provides.

---

## The Implicit-Explicit Spectrum

The four levels of integration strength also reflect how **implicit or explicit** the shared knowledge is:

- **Contract coupling** = most explicit: the interface IS the integration point; changes are visible and versioned
- **Model coupling** = explicit but broader: shared model is known, but the blast radius of model changes spans all consumers
- **Functional coupling** = can be invisible: duplicated business logic in separate components creates a hidden coupling that only surfaces when a spec changes and one side isn't updated
- **Intrusive coupling** = most implicit and fragile: the dependent component uses private internals; the owning team may not know the dependency exists

---

## Connascence Degrees

Within each integration strength level, connascence provides finer-grained comparison. Stronger connascence = more knowledge shared = higher probability of cascading changes within that level.

**Static connascence** (detectable at compile/parse time, weakest → strongest):

- Name → Type → Meaning → Algorithm → Position
- Useful for comparing degrees of contract and model coupling

**Dynamic connascence** (detectable only at runtime, weakest → strongest):

- Execution → Timing → Value → Identity
- Useful for comparing degrees of functional coupling

---

## DDD Tactical Design Connections

- **Aggregate** and **value object** patterns co-locate (low distance) behaviors sharing transactional boundaries (high degree of functional coupling). This is balanced: high strength is offset by low distance.
- Business logic implementation patterns (transaction script, active record, domain model, event-sourced domain model) are chosen based on domain complexity — which directly relates to volatility. Event-sourced domain model is the most complex pattern, appropriate only for core subdomains with highest volatility.

## DDD Strategic Integration Patterns

**Bounded context integration patterns** map directly to integration strength and organizational distance:

- **Ad-hoc patterns** (shared kernel, partnership, conformist) allow more knowledge exchange → higher integration strength. Appropriate when organizational distance is low (same team, close collaboration).
- **Formal patterns** (anti-corruption layer, open-host service/published language, separate ways) reduce knowledge sharing → lower integration strength. Appropriate when organizational distance is high (different teams, independent lifecycles).
- **ACL and OHS** specifically encapsulate internal models behind integration contracts (contract coupling) — the primary mechanism for reducing strength when high distance exists.
