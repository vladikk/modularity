# The Discovery Sweep Catalog

Twelve hunting methods. Each finds a class of coupling the others miss; together they bound the search space well enough that the review's coverage ledger can honestly say what is left. Sweeps 1, 2, 4, 7 form the Standard set (highest yield per token); Exhaustive runs all twelve plus the adversarial second wave.

Every sweep produces **candidates**, not findings — the orchestrator re-verifies each one (Step 3 of the skill) before it may appear in the report.

## Output contract (every sweep, every candidate)

```
- Claim: <the one rule/knowledge shared>
- Operands: <file:line on BOTH sides, with a short quote each>
- Carrier: <import | wire | config | prose | copy | constant | fixture | comment | none(pure duplication)>
- Strength: intrusive | functional | model | contract   (implicit? yes/no)
- Distance: <levels per the model, using the review's distance scale>
- Volatility: <per the owner's ruling / subdomain classification>
- Pin status: <none | pin exists at <file> — VERIFY its operands are live artifacts | partial>
- Suggested severity: Critical | Significant | Minor  (orchestrator may override)
```

Plus per-sweep **denominators** (totals examined / excluded / candidates) for the coverage ledger. If a sweep comes back empty, report the denominators anyway — a verified negative is a result.

## Subagent prompt skeleton

> Read-only reviewer applying the Balanced Coupling model (strength intrusive>functional>model>contract; distance; volatility; BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY). Do NOT modify files. Scope: <paths>. Owner volatility ruling: <ruling>. EXCLUDE these already-known findings: <list/report path>. Your sweep: <paste the sweep section below>. Return candidates in the output contract format, ranked, then your denominators, then a short "checked, clean" list. Be skeptical; no padding — an honest empty result with denominators beats invented findings.

---

## Sweep 1 — Seam map

**Question**: what do the declared integration points share?
**Method**: map imports, ports/interfaces, wire contracts, DB access, events between components. For each pair: what knowledge crosses, at what strength, is it explicit? Use LSP/grep; read the actual call sites, not the architecture docs. Include the composition/wiring layer (DI roots, routers) — rules settle there.
**Finding shape**: model coupling broader than needed; business rules living in transport layers; an ACL that leaks; a "contract" edge whose payload is an internal model.

## Sweep 2 — Declared synchronization markers

**Question**: which pairs *announce* they must stay in sync, and is anything holding them?
**Method**: case-insensitive grep across ALL text (code, tests, IaC, scripts, specs, native code) for: `keep in sync`, `kept in sync`, `in sync with`, `must match`, `must agree`, `must stay`, `stay verbatim`, `verbatim identical`, `mirror`, `mirrored`, `same as the`, `identical to`, `change both`, `both sides`, `lockstep`, `restated from`, `duplicated in`, `copy of the`, `must not drift`, `drift`. Classify every hit: (a) already-known finding, (b) intra-file/rhetorical, (c) a cross-file sync claim → verify a pin exists that reads **both** operands. Report (c) exhaustively with counts for all classes.
**Finding shape**: a "mirrors X" comment with no test reading both sides; a pin that compares a copy to itself (each suite carrying its own copy of a shared vector table is a copy-pin, not a pin).

## Sweep 3 — Undeclared duplication

**Question**: what is duplicated *without* announcing it? (The complement of Sweep 2 — no comment joins the copies.)
**Method**: extract distinctive string literals (≥12 chars) from production source; count occurrences across different top-level owners (different modules, both repo sides, code↔native). Separately: named numeric constants with unusual values — find same-value constants in different owners whose names suggest the same concept. For the top suspects, read both sites and judge same-rule vs coincidence. Spot-probe: pick distinctive multi-word phrases from hot modules' comments/identifiers and grep for second homes.
**Finding shape**: a policy block byte-identical in two prompt files; one regex in two languages; a verification stack re-derived beside the module that exports it; helper clones on security-relevant paths.

## Sweep 4 — Published-decision audit

**Question**: for every decision one side owns, who else re-derives it?
**Method**: enumerate the decisions: enums and status sets, entitlement/gating rules, caps and limits, current-version markers, user-facing copy the server persists, provider/product identifiers, curated vocabularies. For each: grep the *other* side (client for server decisions, code for config decisions, prompts for parser decisions) for literals or logic that re-encode it. Pay special attention to decisions crossing the widest distance (an un-updatable client, an external console, a secret store) and to screens/views that bypass the module that consumed the decision correctly.
**Finding shape**: a UI deriving "premium" from raw status strings the contract says never to re-derive; a client-held "current consent version" the server never compares; copy that must render before the server can supply it, duplicated by hand.

## Sweep 5 — Prose as contract

**Question**: do the normative words match the code?
**Method**: extract behavioral promises from spec descriptions and normative docs — grep for `always`, `never`, `must`, `only`, `will`, `ignored`, `rejected`, `idempotent`, `sorted`, `defaults`, `at most`, `exactly`, `guaranteed`, `tolerant`. Triage to the load-bearing ~25 (what a client author would code against: ordering, replay semantics, error semantics, field presence, resolution orders, units). Verify each against the implementing code on all relevant sides. Also audit the docs' *self-consistency* (a deviation list that is incomplete; a doc contradicting the safety rail it sits beside) and sample doc citations of files/functions/flags for existence.
**Finding shape**: a resolution order the spec states wrong; a promised guard no code implements; a cap stated in the wrong *unit*; a normative example that violates the shipped rule; present-tense instructions about deleted code.

## Sweep 6 — Temporal and semantic constants

**Question**: which rules about time, ordering, and retries must hold *between* owners, and what maintains them?
**Method**: inventory timeouts, TTLs, retry budgets, backoffs, watchdogs, quotas, "per-day" definitions, idempotency horizons, lease durations across all owners (client, server, IaC, native, scripts). For each pair that must satisfy an inequality or agree on a definition: is the relation stated anywhere code-checkable? Check both directions of each seam (e.g., inner timeout × attempts vs the outer gateway timeout; client give-up horizon vs server completion budget; comment claims of "larger than" against the actual values).
**Finding shape**: an LLM call budget exceeding the HTTP gateway timeout in front of it; two "per-day" quotas with two day definitions under one name; a replay window the retrying side cannot know; equal constants under comments claiming strict inequality.

## Sweep 7 — Guard audit ("a guard is not proof")

**Question**: for every pin, drift guard, boundary scan, and CI gate — can it actually go red for the failure that matters?
**Method**: enumerate the guards (contract tests, source scans, drift diffs, CI jobs). For each: (a) identify its two diff/compare operands — are both the *live* artifacts, or is one a copy/the test's own list? (b) read its trigger conditions — CI event types, path filters, label opt-outs, branch scope — and name the change classes that bypass it; (c) check reachability — is the guard itself named in the path filters that decide whether it runs? (d) for lint-style rules cited in code (`disable` comments), confirm the rule is actually enabled.
**Finding shape**: a breaking-change gate that runs only on PRs while pushes to main are possible; disable-comments for a rule set to "off"; a schema guard diffing generated output against itself; a boundary test that polices providers while the protocol goes through allowed methods.

## Sweep 8 — Operational/config shell

**Question**: what does infrastructure know that code also knows?
**Method**: IaC ↔ code: env var names and who reads them; context/flag vocabularies vs the deploy script that forwards them vs the example env file; IAM policies vs the SDK calls actually made; resource-update semantics (does editing config X actually propagate — versioned resources, layers refreshed by presence vs version); alarms/metric filters vs emitted metric names; queue/timeout inequalities vs handler assumptions; secrets' expected JSON shapes vs their in-repo declarations.
**Finding shape**: deploy tooling that cannot set the flags the stacks read; a policy edit that never republishes the version the runtime applies; a model-id default already drifted from what the account can invoke; granted actions with no caller.

## Sweep 9 — Native, build, and vendor carriers

**Question**: what do manifests, plists, build files, and third-party internals encode that code encodes too?
**Method**: platform manifests/entitlements vs the permission/capability requests code makes (derivation claims checked across file boundaries); plugin/package version ranges vs code depending on those packages' *internals* (private class names in manifests, serialization keep-rules, initialization-order assumptions, comments naming plugin bugs); vendor enums/ordinals crossing the system as your model; OAuth/store identifiers restated across source, resources, plists, and consoles; unit/format vocabularies duplicated in native layers.
**Finding shape**: a derived-permissions guarantee that stops at the manifest boundary; intrusive plugin knowledge under a floating caret range; a platform's private enum used as the wire vocabulary with no decoder on the receiving side.

## Sweep 10 — Speculative surface

**Question**: what is exported/published that nothing consumes?
**Method**: for each module barrel/public API: list exports, count external production consumers (word-boundary grep, excluding tests; note that structural typing and relative imports can evade naive counts — check both import forms). Wire schemas: fields the client never reads AND the server only passes through. Channel methods/keys one side never invokes. Providers watched nowhere. Versioned artifacts ("MUST bump") with zero readers of the version. Judge each: deliberate seam (documented) vs accretion.
**Finding shape**: `export *` barrels making the enforceable contract a multiple of the documented one; a "published, versioned vocabulary" whose version never reaches any consumer; required curated data a docstring calls illustrative.

## Sweep 11 — Fractal descent

**Question**: inside the biggest files, which co-located responsibilities share nothing — and which rules are duplicated at distance zero?
**Method**: for each production file over ~800 lines: map responsibility clusters with line ranges; build the shared-helper/shared-state matrix (what each cluster actually calls/touches); find rules re-implemented at multiple sites *within* the file (grep the file for its own repeated shapes: identical epilogues, merge rituals, hand-enumerated field lists, mirror-image helper pairs, side-channel fields whose doc admits a single reader); attribute per-cluster churn from commit subjects. Verdict per cluster pair: separable (low strength, low churn — split candidate) vs entangled (keep; the monolith is balanced — say so). Check whether the architecture docs acknowledge the internal structure or call the file a leaf.
**Finding shape**: a compensation epilogue hand-kept at three sites whose comments cite each other; a write-back protocol restated six times; a 370-line function coordinating seven mutable outcome flags; design docs saying "internal design: not applicable" for a 2,700-line file.

## Sweep 12 — Empirical (git history)

**Question**: what actually changes together, and does the recorded coupling explain it?
**Method**: `git log --name-only` over recent months (exclude merges, generated paths, docs-sync noise, and *bundled sweep commits* — commits touching many unrelated files fabricate pairs; run an "organic" pass without them). Compute cross-boundary co-change pairs (module↔module, repo-side↔repo-side, code↔IaC, code↔native) and per-file churn hubs. For each top pair: explained by a recorded edge (name it) or unexplained → read the two files and identify the forcing knowledge. For churn hubs: is the hub a named hot area, or an unflagged change-amplifier? Interpret with care — co-change frequency is evidence of coupling's *location*, never of domain volatility by itself.
**Finding shape**: a widget file that is the system's #1 churn hub with organic co-change against 15 modules, which no seam-level finding names; a validation that every heavy pair maps to a recorded edge (report that too — it is the review confirming itself).

---

## The coverage ledger

The report's Coverage section states, per sweep: ran/skipped, denominators, and candidates → verified findings. Plus: files ≥150 LOC never read by any sweep (list or count), and anything deliberately out of scope. The ledger is what turns "we looked" into a measurable claim — and tells the next run exactly where to start.

## Dedup and exclusion discipline

Later sweeps receive the merged candidate list of earlier ones as an exclusion list. When two sweeps find the same pair by different methods, keep one finding and note both carriers. When a prior review document exists, treat its findings as the initial exclusion list and verify (spot-audit) a sample of its evidence at HEAD — corrections to the prior record are findings too.
