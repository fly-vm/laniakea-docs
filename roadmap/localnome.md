# Localnome

Localnome is the local build ladder for P1. It is not a second production spec and not a generated package layout. It is the sequence for developing the smallest useful local version of the system, then adding realism one boundary at a time.

The point is to bring iteration forward:

- start with a single Noemar runtime and a fake grounded call;
- add synserv derivation before adding other teleonomes;
- add syngate, attestation, market memory, and chain realism separately;
- only then expand to the five-teleonome and full-cardinality P1 shape.

Localnome docs should stay narrative and decision-oriented until the runtime has a real ingestion format. Do not create placeholder `.synlang` packages, fake boot manifests, or per-Space implementation stubs by guesswork. Container and external-world isolation doctrine lives in [`localnome-containers.md`](localnome-containers.md); it phases container fidelity in only when a rung's success criterion depends on process / identity / workcell separation.

## Carry-Forward Decisions

- Localnome is biased toward developing the local Noemar/runtime path, not toward writing production-candidate teleonome specs.
- A local run may use fake data, fixture feeds, forked chains, local keys, local workcells, and reduced cardinality.
- Localnome must not assume access to production workspaces, production workcells, production endpoints, production keys, or secure production embstate.
- Production candidate teleonome specs are a separate concern; Localnome can use local analogues of them, but should not pretend those analogues are production specs.
- Workspec/workspace parity matters, but the current docs should name the doctrine rather than invent full workspace file layouts.
- Containerization is a fidelity tool, not the starting point. Keep config, keys, logs, and state separable early; require one localtel per container only when Localnome tests the five-actor boundary.
- Details that are not specified should remain open gaps, not placeholder atoms.

The Localnome ladder grows the production synlang additively: each rung's loop body, rules, equations, and atom shapes are production-quality for the scope that rung covers; later rungs extend those bodies at their phase-invariant consumption sites without rewriting earlier work. A black-box-deferred body (real signature, opaque body) at one rung becomes a real body at a later rung; the readers around it do not change.

## Phase Ladder

| Phase | Main question | Added realism | Done when |
|---|---|---|---|
| Localnome 0 | Can Noemar run one grounded callable path? | One runtime, one fake sigil/protocol, one binding, one implement/workcell-shaped adapter. | A deterministic call returns a value and produces enough trace to inspect sigil -> binding -> implement. |
| Localnome 1 | Can synserv derive ER alone? | Real `&core.loop.synserv` heartbeat body and real rollup shape: exobook → riskbook → halobook NFAT projection → structbook → primebook → ER. Source atoms may be fake; attestation gates may be seeded pass; risk-form body is black-box-deferred behind a real signature; missing SDR allocation means `matched = 0`. | One heartbeat produces an evaluated scalar `(prime-er …)` from local atoms through the production-shaped read path and structbook formula. |
| Localnome 2 | Can an external operator mutation cross syngate? | Entity-govops analogue, fake signed envelope, one operational verb such as borrower setup. | Only a valid authorized envelope mutates the target state; invalid variants are rejected. |
| Localnome 3 | Do attestation gates default-deny? | Attestor analogue, pass/missing/stale/fail/scope-mismatch variants. | Only fresh scoped pass attestations enter the rollup path. |
| Localnome 4 | Can market memory reach risk logic? | Market-data analogue, reducer-shaped outputs, quality states; optional fake/source-feed workcell. | Synserv consumes price/liquidity/volatility/quality outputs and follows pinned conservative behavior. |
| Localnome 5 | Can chain reads and receipts be realistic without mainnet authority? | Forked Ethereum/Sky state, `CHAINREAD`-shaped read path, receipt variants; fork-chain or deterministic chain-read workcell. | Reads and receipt checks use production-shaped interfaces against isolated fork/test state. |
| Localnome 6 | Can one NFAT loan flow end to end? | One Halo, one Prime, one borrower, one halobook/riskbook/exobook/structbook path. | Borrower setup, attestation, chain state, market memory, risk form, structbook matching, and ER emission all connect. |
| Localnome 7 | Do the five P1 actor boundaries hold locally? | Synserv, entity-govops, core-govops, attestor, market-data analogues; one localtel container per actor. | The five roles cooperate through syngate without collapsing keys, logs, nonce domains, or authority boundaries. |
| Localnome 8 | Does reduced-cardinality P1 hold together? | One Generator, one Oracle, one Prime, one Halo, one risk class, one halo class, a small loan set; Compose-compatible project as normal host shape. | Reduced P1 runs borrower, attestation, market-memory, SDR, structbook, and ER flows. |
| Localnome 9 | Does full-cardinality local P1 work? | Eight canonical Primes, three Halos, P1 fixed synart topology, all major beacon classes and verbs, local/forked workcells in the project host shape. | Full local topology can run the activation suite against local data, contracts, keys, workspaces, and workcells. |
| Localnome P1 Full | Is the package ready for testosynome rehearsal? | Production-shaped test workspaces, testonome creation material, scenario fixtures, diff expectations. | Outputs can seed P1 testosynome rehearsal without importing production secrets or authority. |

## Container Phasing

The container rule is staged:

- Localnome 0-1: no container requirement; preserve separable config roots, keys, logs, and state.
- Localnome 2: run the syngate boundary for real; containers are optional.
- Localnome 4-5: fake market-feed and forked-chain workcells become useful sidecar candidates.
- Localnome 7: one-localtel-per-container becomes required because this rung tests actor boundaries.
- Localnome 8-P1 Full: a Compose-compatible project becomes the normal localnome host shape.

The fake external world is broader than syngate: syngate intake, `CHAINREAD`, market-feed acquisition / processing, relay receipt/signing surfaces, and attestor/operator fixtures each sit behind their own workcell boundary when that boundary matters.

## Noemar Alignment

Current Noemar code already has the seed of Localnome 0 under older vocabulary:

```text
Protocol.handles       ~= sigil heads
Space.register_protocol ~= binding operation
Protocol.evaluate      ~= implement method
```

For Localnome 0, the most practical path is to use the existing Protocol stack as the first sigil stack proof rather than inventing a new one in docs. The naming mismatch should be explicit: roadmap sigils are conceptual callable names; current Noemar protocol heads are lowercase/prefix-form symbols such as `echo-ping` or `graph-path`.

Localnome 1 should align its executable surface with [`synlang-p1-form.md`](synlang-p1-form.md). Resolving uppercase roadmap sigils versus current protocol prefix rules, first-class binding metadata, and loop/body ingestion is the LN0/LN1 implementation surface.

Open compatibility gaps:

- first-class binding metadata is not yet present;
- uppercase roadmap sigil names do not match current protocol prefix rules;
- workcell/workspace refs are not first-class in the protocol registry;
- boot receipts and package ingestion are not yet specified;
- production/test workspace isolation is roadmap doctrine, not yet a complete Noemar enforcement layer.

## What Not To Generate Yet

Do not add:

- `localtels/` directory trees;
- concrete Compose services or override files;
- placeholder `.synlang` files for every Space;
- pretend Noemar boot manifests;
- invented auth grants, endpoint paths, receipt atoms, or workcell hubs;
- telseed schemas or production teleonome package layouts;
- production-candidate specs inside Localnome.

Those should appear only after Noemar has a real package/ingestion surface or after a specific phase needs a concrete fixture. This does not forbid the minimal real LN0/LN1 ingestion path that the implementation defines; loading one synserv loop body plus seeded source atoms is the work, not speculative package scaffolding.
