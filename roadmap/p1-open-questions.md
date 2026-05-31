# P1 Open Questions

**Status:** Resolved and propagated on 2026-05-22. This file is now a decision record for the ambiguities found before Localnome 1. The selected answers are captured in `../logs/quickstart.md` and have been applied to the canonical roadmap docs.

The "Downstream" note on each item records where the decision was propagated.

---

## A. Load-bearing — block LN1 directly

### A1. Derived-atom lifecycle

**Question.** synserv "re-derives all book state from current input atoms" each heartbeat *and* writes derived atoms (`exobook-current-state`, `custodial-crypto-crr-components`, `riskbook-default-crr`, `nfat-prime-projection`, `structbook-*`, `prime-trrc`, `prime-er`) into book Spaces it doesn't own. The atomspace is append-only. There was no specified supersession rule, no "latest" read convention, no GC story, and no statement about derived-atom retention.

**Decision.** Derived atoms are **frame-local scratch within synserv's heartbeat**. Replay from input atoms + rule atoms (both append-only with epoch keys) is the canonical source of truth for any historical derived value. "Frame-local" explicitly includes in-tick cross-Space visibility — synserv's heartbeat frame is the cross-book duality boundary, so any reader running in the same tick sees the derivation as part of the shared frame context. Off-tick consumers (wardens, audit, late readers) reconstruct historical values by replaying input+rule atoms through synserv. Any persistence layer (rolling-window cache, periodic snapshots) is an implementation optimization sized to measured read patterns, not part of the semantic model.

The atomspace stays append-only for *inputs and rules*. Derived atoms are not persisted as a semantic commitment in P1. Atom traces in the docs show derived values for clarity of derivation flow; they are not claims about what is persisted.

**Downstream.**
- `phase-1-spaces.md` framing and principles — frame-local + replay-canonical derived atom lifecycle.
- `p1-nfat-atom-trace.md` trace discipline — derived atoms shown for derivation logic; replay from input+rule atoms is canonical for historical inspection.
- `phase-1-principles.md` — principle #17 rewritten to frame-local + replay-canonical.

**Status:** Resolved — frame-local scratch with in-tick cross-Space visibility; replay from input + rule atoms is the canonical source of truth; any caching is non-semantic optimization.

---

### A2. "No cross-entart hops in the rollup" is false as written

**Question.** `phase-1-spaces.md` Framing says "Local reads everywhere; no cross-entart hops in the rollup." But the rollup explicitly reads cross-entart in at least three places: riskbook reads `&entity.oracle.crypto-majors.ticks`; structbook reads `&entity.generator.usge.sdr-auction`; structbook reads NFAT projections from each holding-Halo's halobook. `p1-nfat-atom-trace.md` even lists "cross-book duality" as a *sanctioned* cross-Space read mechanism. And cross-book duality itself is named repeatedly but never defined.

**Proposed resolution.** Reword the sentence and define cross-book duality.

**Replacement sentence (Framing bullet, phase-1-spaces.md):**

> **Per-entart local materialization of class/form copies.** Each halo carries its own halo-class + risk-class copies (attestor as sub-Space of the risk class). Each Prime, Halo, and the Generator carry their own `protocol-registry`. **The rollup's cross-Space reads go through four sanctioned mechanisms only: registries, oracle subscription, cross-book duality, and SDR allocation reads.** Per-entart materialization keeps risk-class / halo-class / form copies local so canonical-source propagation can later land additively without rewiring readers.

**Cross-book duality definition (new short subsection in phase-1-spaces.md, before "Per Prime"):**

> **Cross-book duality.** A book *unit* (P1: NFAT halo unit) is the same object viewed from two sides. Issuance writes two atoms: a **liability atom** in the issuing book (e.g., `(nfat-unit …)` and `(nfat-holder …)` in the halobook) and a **holder atom** in the holding Prime's root (`(prime-nfat-allocation …)`). The risk-bearing **projection** value (`(nfat-prime-projection …)`) is derived by synserv from the issuing halobook's input atoms within its heartbeat frame and is visible in-tick to the holding Prime's structbook through that shared frame. The two persistent atoms (liability + holder allocation) are issuer-and-holder-owned; the projection is frame-local per A1 — reconstructable off-tick by replay from input + rule atoms.

This makes the rollup's read topology explicit:

| Reader | Read | Mechanism |
|---|---|---|
| Riskbook risk form | `&entity.oracle.crypto-majors.ticks` | oracle subscription |
| Structbook | NFAT projections in each halobook | cross-book duality |
| Structbook | `&entity.generator.usge.sdr-auction` `(sdr-allocation …)` | SDR allocation read |
| Primebook | `(prime-trc …)`, `(prime-ijrc …)` in own root | local |
| Primebook | `(exsyn-trrc-claim …)` in own primebook | local (patch-beacon writes here) |

**Downstream.**
- `phase-1-spaces.md:14` — replace the sentence.
- `phase-1-spaces.md` — insert "Cross-book duality" subsection.
- `p1-nfat-atom-trace.md` "Trace discipline" — keep the existing mention of cross-book duality as sanctioned; cross-reference the new definition.
- `phase-1-overview.md` Prime–Halo "dynamic auto-wiring" — note that auto-wiring includes the cross-book-duality projection write so the holding Prime's structbook can read the new exobook's projection without follow-up sudo.

**Status:** Resolved — SDR allocation reads are their own fourth explicit mechanism.

---

### A3. What LN1 builds vs. stubs — and reconciling lift with the ladder

**Question.** `localnome.md` LN1 previously described a fake ER-shaped output and listed the risk form as connecting only at LN6. The lift principle says build the real CRR equation in synlang from day 1. These read as contradictory. Separately, "build it once, don't rewrite" (lift) and "add realism one boundary at a time" (the Localnome ladder) needed a one-sentence reconciliation.

**Proposed resolution.**

**LN1 scope, explicit.** Localnome 1 builds:

- the **real** `&core.loop.synserv` heartbeat loop body in synlang;
- the **real** rollup derivation shape (exobook → riskbook → halobook NFAT projection → structbook → primebook → ER), with every read at its phase-invariant consumption site;
- a **black-box-deferred risk-form body** — the `(risk-form custodial-crypto …)` object has a real signature, real `composition-constraints`, and an opaque body that returns seeded CRR component values for LN1's fake exobook. The risk-form body is filled at LN6 without changing the loop body or any read path;
- the **real** structbook matching formula and the **real** ER arithmetic at the top;
- fake source atoms only: fake exobook content, fake market memory (read by the stub risk form), fake attestation gates (always-pass), and missing SDR allocation (treated as zero matched per A4 below).

What LN1 deliberately defers: syngate (LN2), default-deny logic exercise (LN3), real reducer outputs (LN4), real chain reads (LN5), the risk-form body and the end-to-end NFAT flow (LN6), actor boundaries (LN7+).

**Lift ↔ ladder reconciliation (new paragraph in `localnome.md` after Carry-Forward Decisions, and in `roadmap-ideas.md` "The lift principle"):**

> The Localnome ladder grows the production synlang **additively**: each rung's loop body, rules, equations, and atom shapes are production-quality for the scope that rung covers; later rungs *extend* those bodies at their phase-invariant consumption sites without rewriting earlier work. A black-box-deferred body (real signature, opaque body) at one rung becomes a real body at a later rung; the readers around it do not change. This is the same shape as insyn/exsyn ("the synlang code doesn't change at the migration boundary") applied across LN rungs instead of across phases.

**Downstream.**
- `localnome.md` LN1 row — expand "Added realism" and "Done when" with the explicit scope above.
- `localnome.md` — add the lift ↔ ladder paragraph.
- `roadmap-ideas.md` "The lift principle" — add the same paragraph (cross-link).
- `phase-1-overview.md` "Read first" — add "LN1 builds the real loop body; the risk-form body is the canonical black-box deferral for LN1, filled at LN6."

**Status:** Resolved — LN1 builds the real synserv loop and ER rollup shape, with the risk-form body black-box-deferred.

---

### A4. The structbook capital formula — `forced-loss-capital`, `default-CRR` vs `RW`, worked-example numbers

**Question.** The structbook formula used `forced-loss-capital`, which the risk form did not define consistently. `phase-1-principles.md` previously used RW wording while formula files used `default-CRR`, and the worked examples had mismatched illustrative numbers.

**Proposed resolution.** **Pin three things.**

1. **`forced-loss-capital := spread-CRR + liquidity-CRR`** at the position level. Both components express realized losses if the Prime is forced to sell before maturity (MTM spread loss + wrapper-exit liquidity loss); they're additive because they hit a single sale. Rate-CRR is separate (carry/funding mismatch over remaining TTM) and is added to the structbook formula as its own term, not folded into forced-loss-capital.

2. **Matched multiplier is `default-CRR`,** not `RW`. `default-CRR` is the position-level fundamental-loss number the risk form computes (worst stressed senior loss). `RW` is an *asset-profile input* that feeds into the scenario stress; it's not a separate position-level multiplier. `phase-1-principles.md` #6 and #7 are reworded:
   - #6: "**Matched positions are capitalized for the position's default-CRR only** — held-to-par matching covers spread, rate, and liquidity for the matched portion."
   - #7: "**Unmatched positions are capitalized for `max(default-CRR, forced-loss-capital)` plus rate-CRR,** where `forced-loss-capital = spread-CRR + liquidity-CRR`."

3. **Canonical structbook formula (single source of truth, in `capital-formula.md`):**
   ```
   forced-loss-capital := spread-CRR + liquidity-CRR
   matched   := min(position_size, available_structural_demand_capacity_at_required_bucket_or_higher)
   unmatched := position_size − matched
   position_capital := matched   × default-CRR
                     + unmatched × max(default-CRR, forced-loss-capital)
                     + unmatched × rate-CRR
   ```

   The other lean files (`matching.md`, `custodial-crypto-risk-form.md` §6, `risk-framework.md` capital formula) state this formula in the same form and link to `capital-formula.md` for the definition of `forced-loss-capital`.

4. **Worked-example numerical fix.** `phase-1-spaces.md` and `p1-borrower-nfat-user-scenario.md` use the same illustrative components: default-CRR 0.055, spread-CRR 0.018, rate-CRR 0.012, liquidity-CRR 0.030, and forced-loss-capital 0.048.

**Missing-SDR-allocation behavior.** When `(sdr-allocation prime bucket amount E)` for the required-or-higher bucket is absent, `available_structural_demand_capacity` is treated as zero, the position is fully unmatched, and the formula evaluates as the unmatched branch. (This is what the borrower scenario §11 does; pin it explicitly so LN1's missing-atom semantics are unambiguous.)

**Downstream.**
- `capital-formula.md` — promote to canonical definition site for `forced-loss-capital`; rewrite §2 with the formula block above.
- `matching.md` §3 — restate the formula identically; link to `capital-formula.md` for `forced-loss-capital`. Also drop the misleading old SDR-availability eligibility clause in §2 (see B5).
- `risk-framework.md` capital-formula section — restate identically.
- `custodial-crypto-risk-form.md` §6 — restate identically; spell out `forced-loss-capital`.
- `phase-1-principles.md` #6 and #7 — reword as above.
- `phase-1-spaces.md` worked example steps 3 and 6 — renumber the components so step 3's scenario-loss number is default-CRR and step 6 uses the same number; supply explicit spread/liquidity/rate values.
- `p1-nfat-atom-trace.md` §3.4 — confirm the structbook formula matches.

**Status:** Resolved — `forced-loss-capital = spread-CRR + liquidity-CRR`; matched uses default-CRR; unmatched adds rate-CRR.

---

### A5. No executable synlang exists in the corpus

**Question.** Every code block in roadstart + roadmap is a data atom or prose pseudocode. There is not one example of an executable rule, loop body, or risk-form equation. The `(risk-form …)` object is half-specified (only `composition-constraints` filled in custodial-crypto; `variables`, `equation-*`, `resolution-tier` named but never instantiated). LN1's deliverable *is* writing the synserv loop body in synlang. `localnome.md` "Noemar Alignment" already flags compatibility gaps between roadmap sigil naming and current Noemar protocol naming.

**Proposed resolution.** A small new doc — `synlang-p1-form.md` — pins the minimal executable-synlang form LN1 needs. It does not specify the full language; it commits to the surface LN1 + LN6 require:

- **Atom syntax** — already in `grounding-and-workcells.md` §1; cross-reference.
- **Rule form** — `(rule {rule-id} (reads [&space …]) (when {predicate}) (yields {atom-template}))`; the predicate uses stdlib + special forms + variables; `yields` is a heartbeat derivation atom template that the runtime materializes per match. One worked example: the rollup-gate rule.
- **Loop body form** — `(loop-body (loop-step …) (loop-step …) …)` where each `loop-step` is a special-form-wrapped sequence of reads, derivations, and writes. One worked example: the synserv heartbeat skeleton (settlement-level → exobook level → riskbook level → halobook level → structbook level → primebook level → emit `prime-er`).
- **Equation form for risk forms** — fills the `variables` / `equation-default` / `equation-m2m` / `equation-htm` / `resolution-tier` slots `risk-framework.md` names. One worked example: the custodial-crypto `equation-default` skeleton with composition-constraints already in place and the body still black-boxed (LN1) or filled (LN6).

This is the smallest commitment that lets LN1 be written without ad-hoc invention. The doc explicitly defers anything the runtime doesn't yet need (pattern matching beyond simple equality, recursion beyond `(run-forever)`, type system).

**Downstream.**
- New `synlang-p1-form.md` in `roadmap/`.
- `roadstart/README.md` — add to reading order between `phase-1-principles.md` and `attestor-atom-schema.md` (everything below it implicitly depends on this form).
- `localnome.md` "Noemar Alignment" — link the new doc; restate that resolving the uppercase-vs-prefix and binding-metadata gaps is the LN0/LN1 implementation surface.
- `custodial-crypto-risk-form.md` §1 — extend the `(risk-form …)` object with placeholder `variables` / `equation-default` / `resolution-tier` slots so the LN1 stub has a complete object to fill at LN6.

**Status:** Resolved — add `synlang-p1-form.md` as the minimal roadmap syntax/form doc.

---

## B. Atom-schema contradictions across the worked examples

These are mechanical fixes once decided; no deep design call needed.

### B1. `prime-trc` arity

**Question.** `phase-1-spaces.md` (root row + genesis step 14) wrote 2-arg `(prime-trc {prime} {amount})`. `p1-nfat-atom-trace.md` §2.2 needed 3-arg `(prime-trc Prime1 {trc} E)`.

**Proposed resolution.** **3-arg** `(prime-trc {prime} {amount} {epoch})`. Epoch stamp gives provenance for governed P1 updates and supports latest-`E` reads. Update `phase-1-spaces.md` root row and genesis step 14. Same atom shape as `prime-ijrc` (see B2).

**Status:** Resolved and propagated.

---

### B2. `prime-ijrc` is read but not declared

**Question.** The SDR auction reads "each Prime root's IJRC atom" and `p1-nfat-atom-trace.md` §2.2 reads `(prime-ijrc Prime1 {ijrc} E)`, but `prime-ijrc` is not listed in the Prime root's "Holds" and not written in genesis step 14.

**Proposed resolution.** Add `(prime-ijrc {prime} {amount} {epoch})` (3-arg, same shape as `prime-trc` per B1) to:
- `phase-1-spaces.md` Per-Prime root row — "Holds: identity, auth, sub-space registry, epoch-stamped `(prime-trc …)` atom, epoch-stamped `(prime-ijrc …)` atom, per-Prime relay/operator config."
- `phase-1-spaces.md` genesis step 14 — write both `(prime-trc …)` and `(prime-ijrc …)`.

Independent question resolved: P1 carries `(prime-trc …)` (aggregate TRC scalar) + `(prime-ijrc …)` (IJRC tier scalar) only; EJRC and SRC tiers are deferred until P1 logic actually reads them.

**Status:** Resolved and propagated.

---

### B3. Riskbook-level CRR atom shape, and aggregation of non-default components

**Question.** `p1-nfat-atom-trace.md` §3.2 emitted `(riskbook-default-crr rbk H {value})` (default only), while `p1-borrower-nfat-user-scenario.md` §9 previously emitted a riskbook-level all-component CRR atom. `custodial-crypto-risk-form.md` §5 only specified default-CRR aggregation; spread/rate/liquidity aggregation was unspecified.

**Proposed resolution.** **Per-exobook CRR components are the only CRR atoms; riskbook-level aggregation exists for default-CRR only.** The structbook reads per-exobook components via the halobook's `(nfat-prime-projection …)` (one projection per NFAT, which has one source exobook). The borrower scenario's old riskbook-level all-component atom is removed; its four-component data is what's already in the per-exobook `(custodial-crypto-crr-components $exobook H …)` atom that the trace defines.

Canonical riskbook-level derived value (per A1, frame-local — shown here for derivation clarity, not as a persisted atom shape):
```
(riskbook-default-crr {rbk} {H} {value})
```

If a future risk form genuinely needs riskbook-level spread/rate/liquidity (because some structural treatment aggregates them), that aggregation gets added at that form's introduction — not speculatively in P1.

**Downstream.**
- `p1-borrower-nfat-user-scenario.md` §9 — uses only the per-exobook `(custodial-crypto-crr-components spark-term-loan-001 H (default-crr 0.055) (spread-crr 0.018) (rate-crr 0.012) (liquidity-crr 0.030) (binding-scenario btc-liquidity-crash-v1))`.
- `custodial-crypto-risk-form.md` §5 — add one sentence: "Spread, rate, and liquidity-CRR are consumed per-exobook at the structbook level; only default-CRR is aggregated to riskbook level in P1."

**Status:** Resolved and propagated.

---

### B4. `prime-er` stored value form

**Question.** `p1-nfat-atom-trace.md` §3.5 previously showed `prime-er` as an unevaluated s-expression, while `p1-borrower-nfat-user-scenario.md` §12 showed `(prime-er Prime1 0.31005 H)` as an evaluated scalar.

**Proposed resolution.** **Evaluated scalar.** The atom trace's `(/ (+ …) …)` form is symbolic notation showing the derivation; the value that flows between readers (in-tick or via replay) is the scalar. State explicitly in the trace discipline section that the atom trace uses s-expressions to show how a value is derived, and the data shape that readers consume carries the resulting scalar. (Per A1, the value is frame-local; this resolution is about shape, not persistence.)

This also resolves a small homoiconicity question: P1 atoms persist *values*, not unevaluated code. Code blobs are materialized at bootstrap and bound to sigils; rule/loop/equation bodies live in dedicated Spaces (`&core.loop.synserv`, risk-class Spaces, etc.) as program text the runtime evaluates — but ordinary atoms in book Spaces are data.

**Downstream.**
- `p1-nfat-atom-trace.md` "Trace discipline" — add one sentence on s-expression-as-derivation-notation vs. scalar-as-stored-atom.
- `p1-nfat-atom-trace.md` §3.5 — rewrite the `prime-er` example as a scalar derived from named scalars, e.g., `(prime-er Prime1 {prime-er-value} H)`.

**Status:** Resolved and propagated.

---

### B5. structbook eligibility vs. zero-SDR membership

**Question.** `matching.md` §2 previously made SDR availability an eligibility gate. `p1-nfat-atom-trace.md` §4 and `p1-borrower-nfat-user-scenario.md` §11 kept a zero-SDR-allocation position *in* structbook as fully unmatched. The zero-SDR path is the natural LN1 happy path, so the contradiction directly hit LN1.

**Proposed resolution.** In P1, structbook eligibility = SPTP defined (which means it's term-matchable in principle, and structbook is the only active P1 sub-book anyway). SDR-allocation availability determines matched-vs-unmatched *within* structbook, not membership.

Replace `matching.md` §2 with:
```
P1 structbook eligibility:
   has-sptp(position) → routes to structbook
SDR-allocation availability determines the matched portion within structbook
(missing or zero allocation → matched = 0, unmatched = full position).
```

**Downstream.**
- `matching.md` §2 — rewrite as above.
- `p1-nfat-atom-trace.md` §3.4 — confirm the `structbook-eligibility` atom no longer treats `sdr-allocation-present` as a gate; it's surfaced for observability but not part of the pass/fail decision.

**Status:** Resolved and propagated.

---

## C. Lower-priority, but worth shoring up before git

### C1. Prime topology

Decision: P1 uses canonical IDs `Prime1` through `Prime8`. Given names are separate atoms. Current known mapping is `Prime1 spark`, `Prime2 grove`, `Prime3 obex`, `Prime4 keel`, `Prime5 skybase`, `Prime6 launch6`; `Prime7` and `Prime8` have no given names yet.

### C2. LN1 missing-atom semantics

Pin in `localnome.md` LN1 row (or in a new "P1 missing-atom semantics" appendix referenced by LN1):
- Missing `(sdr-allocation …)` for required-or-higher bucket → matched = 0 (per A4 / B5).
- Missing `(exsyn-trrc-claim {prime} …)` → exsynTRRC contribution = 0.
- Missing `(borrower-readiness-attestation …)` → can't submit borrower-inclusion request (LN1 sidesteps; LN3 tests).
- Missing `(custodial-borrower-admission …)` / `(riskbook-attestation …)` / `(exobook-term-attestation …)` → exclude from rollup (LN3 tests; LN1 always feeds `pass` so doesn't exercise).

### C3. TRC mutability inside P1

Decision: use epoch-stamped `(prime-trc {prime} {amount} {epoch})` and `(prime-ijrc {prime} {amount} {epoch})`; governed P1 updates are allowed without changing topology.

### C4. Naming slips

- `big-picture.md` naming example uses the real sub-kind form (`&entity.halo.spark-term.halobook.hbk-001`).
- "Content-addressed names" (commitment #3) reads as universal but constructor-made book ids are `hbk-001` / `rbk-001` style. Clarify scope: content-addressing applies to implement code blobs and risk-form imports (which carry hashes); book ids are human/op-readable and the content-addressed binding is the registry atom (`(sub-space …)` + parent pointers).

**Flag for first-principles decision** — not auto-fixing.

### C5. Glossary gap — IJRC / EJRC / SRC / MDC

Never expanded in roadstart/roadmap. Recommend adding a one-line glossary in `risk-framework.md` "Capital stack" section: IJRC = Initial Junior Risk Capital, EJRC = External Junior Risk Capital, SRC = Senior Risk Capital, MDC = Mezzanine Default Capital (verify these expansions against the canonical `accounting/capital-stack.md`). Also fix `capital-formula.md` which writes "JRC" where it means IJRC.

### C6. Ingestion format and the "don't generate yet" rule

`localnome.md` "What Not To Generate Yet" forbids placeholder `.synlang` packages and boot manifests, but LN1 cannot run without *some* way to load a loop body + fake atoms into Noemar. **Resolution:** add a sentence that explicitly distinguishes "doc placeholders for imagined infrastructure" (forbidden) from "the minimal real ingestion path LN0/LN1 implementation defines" (encouraged — that's the work). The boot manifest shape in `grounding-and-workcells.md` §5 is fine as a *target*; LN1 implements only the slice it needs (a single synserv loop body + a way to seed source atoms into a test Space).

---

## Decision summary

| ID | Type | Status |
|---|---|---|
| A1 | derived-atom lifecycle | Resolved — frame-local scratch + in-tick cross-Space visibility; replay from input+rule atoms is canonical |
| A2 | cross-entart / cross-book duality | Resolved — four mechanisms: registries, oracle subscription, cross-book duality, SDR allocation reads |
| A3 | LN1 scope + lift↔ladder | Resolved — real loop + real ER rollup shape; black-box risk form body |
| A4 | capital formula & forced-loss-capital | Resolved — `forced-loss-capital = spread-CRR + liquidity-CRR`; matched × default-CRR |
| A5 | executable synlang form | Resolved — added `synlang-p1-form.md` |
| B1 | `prime-trc` 3-arg | Resolved — epoch-stamped |
| B2 | `prime-ijrc` declaration | Resolved — epoch-stamped root atom |
| B3 | per-exobook only CRR | Resolved — riskbook aggregates default-CRR only |
| B4 | `prime-er` scalar | Resolved — persisted scalar |
| B5 | structbook eligibility | Resolved — SPTP routes to structbook; SDR controls matched amount |
| C1 | Prime topology | Resolved — `Prime1` through `Prime8`, given names separate |
| C2 | LN1 missing-atom semantics | Resolved — explicit zero/default behaviors |
| C3 | TRC mutability inside P1 | Resolved — epoch-stamped, governed P1 updates allowed |
| C4 | naming slips | Resolved — examples use halobook/riskbook/exobook naming |
| C5 | glossary | Resolved — capital-tier acronym glossary added |
| C6 | ingestion vs. don't-generate-yet | Resolved — minimal real LN0/LN1 ingestion path allowed |

These answers have been propagated into `phase-1-spaces.md`, `phase-1-principles.md`, the worked examples, the lean risk/matching/capital files, `synlang-p1-form.md`, `localnome.md`, and `roadmap-ideas.md`.
