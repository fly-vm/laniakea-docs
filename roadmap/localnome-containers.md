# Localnome Containers

**Scope:** Container and external-world isolation doctrine for Localnome. This is not a Compose spec, not a package layout, and not a telseed schema. It explains how container fidelity should enter the Localnome ladder without pulling implementation ahead of Noemar's package / ingestion surface. Boundary principles for Noemar, synlib, and telseed live in [`noemar-synlib-telseed.md`](noemar-synlib-telseed.md).

Localnome remains the local build ladder: start with the smallest useful runtime proof, then add realism one boundary at a time. Containers are part of that realism, but only when the boundary being tested is process / identity / workcell separation.

## 1. Isolation model

Isolation is a fidelity requirement. A green Localnome run only means something when the rung under test preserves the boundary it claims to test.

Use four separate layers:

| Layer | Meaning |
|---|---|
| **Localnome host unit** | The whole local ensemble. Later this is naturally a Compose project or equivalent project namespace. |
| **Localtel isolation** | One independently-operating local teleonome analogue: its own keys, nonce domain, logs, filesystem state, and network identity. Later this maps to one OCI container per localtel. |
| **Workcell isolation** | Bounded operational resources such as chain RPC, signer, market feed, data processing, attestor documents, or operator UI. These can run as sidecars or adjacent services and should own dangerous resources. |
| **Noemar frame isolation** | In-runtime state snapshots / forks. These are not Docker or container isolation; they are runtime state mechanics. |

Do not collapse these layers. A Compose project is not a Noemar frame. A workcell sidecar is not a localtel. A localtel container is not the whole localnome.

## 2. Container principles

- **One OCI container per localtel when actor boundaries are under test.** This is the clean local approximation of independent embodiments: separate process space, filesystem, logs, network identity, keys, and nonce domains.
- **No Docker-in-Docker.** Nested container daemons require broad privilege and undermine the authority-isolation model. Run multiple localnomes as multiple project namespaces, not as containers that start their own privileged daemon.
- **Project namespace is the ensemble boundary.** A Compose-compatible project gives a localnome its own network and volume namespace and can start / stop the ensemble as a unit. The implementation should remain compatible with Podman, Docker Engine, or Colima-style Docker CLI setups.
- **Sidecars are workcells.** Put dangerous or externally-real resources behind workcell boundaries: forked chain, signer, market-data source, attestor document fixture, operator UI, or data-processing hub. The localtel talks to the workcell interface; it does not own the dangerous resource directly.
- **Develop locally, soak on remote Linux.** Treat the localnome as source plus artifacts in a repo, not a blob. Local development keeps the fast edit loop; remote Linux runs longer-lived reduced/full localnomes. Architecture mismatches should be solved by building on the target host or by multi-arch builds when that becomes necessary.

These are hosting principles, not current file-generation instructions. Do not introduce concrete Compose services, endpoint names, volume names, or generated directories until a specific Localnome rung needs them.

## 3. Phasing into Localnome

Container fidelity should arrive only when it tests a real boundary.

| Rung | Container stance |
|---|---|
| **Localnome 0-1** | No container requirement. Keep config roots, key material, logs, and state separable so later containerization is straightforward. |
| **Localnome 2** | Real syngate boundary appears. Containers remain optional; the important test is signed-envelope ingress and rejection behavior. |
| **Localnome 3** | Attestor behavior can still be local or fixture-backed. Preserve separate attestor key/log/state paths. |
| **Localnome 4** | Market-data realism starts. A fake/source-feed workcell or data-processing sidecar becomes useful, but not mandatory if it slows the loop. |
| **Localnome 5** | Forked Ethereum / `CHAINREAD` realism starts. A fork-chain or deterministic chain-read workcell is the first strong sidecar candidate. |
| **Localnome 6** | End-to-end NFAT flow may still run reduced-cardinality in a simple host shape, as long as syngate, chain, market, and attestor boundaries are not semantically collapsed. |
| **Localnome 7** | First mandatory one-localtel-per-container rung. This phase asks whether the five P1 actor boundaries hold locally; container isolation is part of the acceptance criterion. |
| **Localnome 8-P1 Full** | Compose-compatible project becomes the normal host shape for reduced/full local P1, with sidecars for test/fake workcells. |

The practical rule: add container machinery the moment a rung's success criterion depends on machine-like separation; do not add it earlier as infrastructure decoration.

## 4. Fake external world

"Fake internet" is not only syngate.

From the synserv runtime's P1 view, the ordinary workcell-backed sigils are only `SYNGATE-READ` and `CHAINREAD`. But the full localnome includes teleonome-local workcells that sit outside synserv's callable inventory. External-world fakes should therefore be described by boundary, not by one sigil.

| Boundary | Localnome treatment |
|---|---|
| **Syngate intake** | Run the signed-envelope path for real once Localnome 2 begins. Localtels submit envelopes; synserv performs the second-pass auth/routing logic. |
| **Chain reads** | Use a forked Ethereum / Sky state or deterministic fixture with the same `CHAINREAD`-shaped interface. The synlang read path should not change. |
| **Market data** | Use fake/source-feed and data-processing workcells that emit reducer-shaped outputs and quality states. Market data is not just final tick injection. |
| **Relay / govops action** | In local/test settings, use signer / receipt / fork-chain sidecars where needed. P1 synserv still does not gain `SENDTX`; relay actions remain external/operator decisions recorded as signed atoms. |
| **Attestor / operator input** | Use fixture documents, scripted test UI, or bounded human-input workcells when the scenario requires operator ceremony. These are local teleonome workcells, not synserv sigils. |

Internal localtel-to-synserv communication should become real hub-and-spoke network traffic when actor boundaries are under test. Fake the external world behind workcells; do not fake the gate path that is itself under test.

## 5. What Localnome must teach telseed

A full teleonome container plan is necessary later. It belongs with the telseed design: the package of Noemar, synlang, synart / telart / embart seed material, synlib refs or snapshots, workspecs, and enough installer logic for a teleonome embodiment to install and operate its workcells.

Do not decide that package shape here. Localnome should generate the evidence for it. By Localnome 7 through Localnome P1 Full, the work should clarify:

- what belongs in an image vs an artifact bundle;
- how Noemar, synlang source, synart references, telart seed state, embart seed state, workspecs, and installer logic are separated;
- how workcell sidecars are declared without embedding secrets or production endpoints;
- how local / test / production bindings differ without changing the core teleonome code path;
- how keys, nonce domains, logs, secure embstate, and production workcells remain outside forkable artifacts;
- how a production teleonome can produce testonomes without cloning authority.

The later telseed / container package spec should be a conclusion drawn from these Localnome rungs, not a schema invented before the runtime and local actor boundaries have paid rent.

## 6. Non-goals

Do not add yet:

- `localtels/` directory trees;
- concrete Compose services or override files;
- endpoint, volume, image, or network names;
- pretend Noemar boot manifests;
- telseed schema fields;
- invented auth grants, receipt atoms, workcell hubs, or package commands.

Those should appear only after Noemar has a real package / ingestion surface or after a specific Localnome rung requires concrete fixture material.
