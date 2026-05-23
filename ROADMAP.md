# AiOSComms — Roadmap

The time-ordered delivery plan for the AiOS communication widget.

Phases use the **AiOS project's global phase numbers (P0–P7)** so they line up
with the parent `PLAN.md`, `DECISIONS.md`, and `TODO.md`. Milestones **M1 / M2 /
M3** are the AiOS project milestones — AiOSComms does not set its own.
`X-*` codes reference [FEATURES.md](FEATURES.md); `F-*` codes reference the
parent `FEATURES.md`.

Dates are targets. The standing posture, inherited from the parent project, is
to **compress scope rather than miss a date** — and for this component the
sharpest compression lever is **cutting providers, never weakening the
untrusted-content boundary.**

## Where AiOSComms sits relative to the milestones

AiOSComms is **not on the M1 critical path** — M1 is AiOSCanvas booting a
canvas with one widget, with no AI. AiOSComms is a Phase-2-onward component: it
needs the agent runtime, AiOSVault's credential serving, and (for the product
form) AiOSCanvas's widget contract. Its useful early life is as a **headless,
dogfoodable email core** that lands before any of the canvas/agent integration,
so value arrives without waiting on the whole stack.

## Milestone overview

| Milestone | Date | Phases | What AiOSComms delivers |
|-----------|------|--------|--------------------------|
| **M1 — Booting prototype** | Q3 2026 | — | Nothing required. AiOSComms is post-M1; at most, Phase-0 scaffolding may begin opportunistically. |
| **M2 — Daily driver** | Q4 2027 | P0–P4 | A headless communication core with **Gmail + generic IMAP/SMTP** and **Telegram**; the untrusted-content boundary; AiOSVault-mediated credentials; the versioned consumer interface; the unified-inbox canvas widget. Jason's personal email + one chat network, agent-triaged. |
| **M3 — Alpha release** | Q4 2028 | P5–P7 | Breadth among sanctioned providers (Matrix, Discord-bot, Outlook/Graph); voice-ready unified pane; the AiOSOrg-facing interface hardened; red-team pass on the injection surface. |

## P0 — Foundations & Scoping · *Comms involvement: heavy*

**Goal:** turn the bare repository into a buildable project and lock the
scoping decisions this component lives or dies on.

- Scaffold the Cargo workspace (recommended: `aioscomms-core` + `aioscomms`),
  `rust-toolchain.toml`, `Makefile`, `NOTICE`, CI (`fmt` / `clippy` / `test` /
  `cargo audit` / `cargo deny`), `.gitignore` already present.
- **Confirm the scoping decisions** from [PLAN.md](PLAN.md): the Rust-core +
  Python-client stack; the integration order (email → Telegram → breadth); the
  per-connector "process boundary, never linked C in the core" policy; crate
  layout; the encrypted store choice.
- **Design `DESIGN.md`** — the `Provider` connector trait; the unified
  Account/Thread/Message/Contact schema; the encrypted store schema; the
  untrusted-content boundary (reader contract, sanitizer policy); and **the
  consumer-interface schema + its versioning policy** (load-bearing for AiOSOrg).
- Write the component **`THREAT_MODEL.md`**, extending parent THREAT_MODEL §7.5
  (hostile external content) with the connector- and parser-level specifics.

**Exit:** `make build` / `make test` / `make lint` succeed in CI; the design,
the consumer-interface schema, and the threat model are written down; the
integration matrix is confirmed with Jason.

## P1 — Email Core (headless) · *heavy*

**Goal:** a working, **dogfoodable headless email engine** — the whole
architecture exercised end-to-end on the two Green email providers, with no
canvas or agent dependency yet.

- `X-MODEL` — the unified Account/Thread/Message/Contact/Attachment model.
- `X-STORE` — the encrypted-at-rest local store (messages, contacts,
  attachments, sync cursors).
- `X-CONNECT` — the `Provider` trait, and the **Gmail (API + OAuth)** and
  **generic IMAP/SMTP** connectors.
- `X-SYNC` — fetch / normalize / dedupe / thread; read & flag state.
- `X-SAFETY` — the **untrusted-content boundary**: MIME parsing, HTML
  sanitization to an inert subset, the unprivileged-reader contract producing
  structured safe summaries. Built *here*, with the first connector, not later.
- A minimal **development UI** (or CLI) so the email core is usable before the
  canvas widget exists.

**Exit:** Jason can connect a Gmail account and an IMAP/SMTP account headlessly,
read a unified threaded inbox, and send — with every inbound body passing
through the untrusted-content boundary. **Credentials come from AiOSVault** (this
is the first hard dependency: needs AiOSVault able to store/serve the
credential; OAuth mediation prefers AiOSVault V2).

**Depends on:** AiOSVault credential storage. Not on AiOSCanvas or the agent.

## P2 — Consumer Interface & Agent Seam · *moderate*

**Goal:** make the email core drivable by the agent and (later) AiOSOrg, once
the parent local-AI backbone (parent Phase 2) exists.

- `X-IF` — the **consumer interface**: Unix-socket JSON-RPC + event stream
  (read, subscribe, send, mark/flag, `extract_actionable`), with `Caller`
  identity and **schema versioning from this first cut**.
- `X-AGENT` — the agent-facing contract: the planner sees only safe summaries;
  per-context tool allowlists (reading context cannot send/delete); provenance
  and audit hooks (F-AUDIT); the outbound undo window (F-RECOVERY).
- `python/` — the `CommsClient` over the control socket (the agent/AiOSOrg seam).

**Exit:** the agent can triage and (gated) send through the consumer interface;
`extract_actionable` returns structured candidate actions as inert data.

**Depends on:** the parent agent runtime existing; AiOSVault V2 for mediated
token APIs. First phase coupled to a sibling deliverable — sequence accordingly.

## P3 — Identity & Security Surfaces · *light*

**Goal:** align the store and the consumer interface with the AiOS identity and
storage model as it lands.

- Bind the encrypted store to AiOS **F-STORAGE** (per-object keys / crypto-erase)
  rather than a component-local scheme, and confirm AiOSFSS syncs the ciphertext
  store cleanly (state dir kept outside the synced tree).
- Carry the **user scope** end-to-end (accounts, threads, messages, contacts)
  for the parent's multi-user-ready posture.
- Tie account-credential access to identity state (locked vault ⇒ no fetch/send).

Minimal phase for AiOSComms; most P3 effort is in the parent, AiOSVault, and
AiOSFSS.

## P4 — Unified Pane & First Chat Provider · *heavy* · **→ M2**

**Goal:** the AiOS milestone M2 contribution — the product form (the unified
canvas pane) and the first chat network, proving the model spans email *and*
real-time chat.

- `X-WIDGET` — the **unified-inbox / conversation canvas widget**, rendered into
  AiOSCanvas's widget contract; touch-first triage, compose, provenance/safety
  cues, agent-drafted-reply markers + undo. (Depends on AiOSCanvas's widget
  model — built after the headless core, like AiOSTerminal's canvas frontend.)
- `X-CONNECT` — the **Telegram** connector (Bot API and/or MTProto user session
  per the confirmed ToS scope), exercising push streams, presence, and a
  non-MIME message shape against the unified model and the safety boundary.
- Voice-ready triage and read-aloud with bystander-aware output (parent
  F-VOICE), given message sensitivity.

**Exit (= M2 contribution):** with the agent runtime and AiOSCanvas, the user
has a unified pane over Gmail + IMAP/SMTP + Telegram, agent-triaged, usable as
part of the daily driver.

**Depends on:** AiOSCanvas widget contract (for the widget); the agent runtime;
AiOSVault V2.

## P5 — Provider Breadth · *moderate* · contributes to M3

**Goal:** widen to the rest of the **sanctioned** provider set — never the Red
ones.

- `X-CONNECT` — **Matrix** (`matrix-rust-sdk`, open standard), **Discord**
  (bot scope only — never user/self-bot), **Outlook / Microsoft Graph** (second
  email provider).
- Per-connector capability advertisement so the unified pane and consumer
  interface degrade gracefully across providers.
- Re-evaluate the deferred set (Signal, SMS-bridge) against legal/maintenance
  cost and the existence of the parent phone-companion item — **revisit, build
  only if justified.**

**Explicitly still out of scope:** WhatsApp personal accounts (no lawful path),
Discord-as-user / self-bots (ban-on-sight), RCS direct.

## P6 — AiOSOrg-Facing Hardening & Rich Content · *moderate* · contributes to M3

**Goal:** make AiOSComms a solid backbone for the organizational widget and
render rich content safely.

- Harden and finalize the **consumer interface** against AiOSOrg's actual needs
  once AiOSOrg planning has happened — freeze the versioned schema for `1.0`.
- Sharpen `extract_actionable` (event proposals, deadlines, task implications)
  as inert structured data feeding AiOSOrg.
- In-canvas rich rendering of messages/attachments via the parent **F-INTEROP**
  sandboxed-parser path (PDF/DOCX/image previews) — sanitized, never
  auto-executed.
- Contact-unification policy across providers (privacy-reviewed).

**Depends on:** AiOSOrg's own planning starting (for the interface freeze);
parent F-INTEROP sandboxed parsers.

## P7 — Hardening & Polish · *moderate* · **→ M3**

**Goal:** the AiOS milestone M3 — alpha-quality communication widget.

- **Red-team the injection surface.** AiOSComms is a primary subject of the
  parent P7 red-team exercise — adversarial emails/messages against the
  untrusted-content boundary, the planner/reader split, and cross-domain
  confirmation.
- Performance under real mailbox/chat volume (large mailboxes, busy channels);
  sync robustness and conflict handling across devices via AiOSFSS.
- Accessibility and recovery UI; provider-token expiry / re-auth flows polished.
- External review of the connector credential paths (the raw-handoff fallbacks
  especially), coordinated with the AiOSVault review.

## Cross-component dependencies

- **AiOSVault — hard, early.** No account connects without a vault-held
  credential (P1); mediated token APIs (Gmail/Graph/Telegram-bot/Discord/Matrix)
  need **AiOSVault V2** (P2). AiOSVault V2 is itself the AiOS M2-gating piece, so
  a vault slip cascades into AiOSComms's M2 contribution.
- **Parent agent runtime — P2 onward.** The consumer interface and the safety
  model presuppose the agent runtime exists.
- **AiOSCanvas — P4 onward (widget only).** The unified-pane widget needs
  AiOSCanvas's widget/surface model. The headless core (P1–P2) does **not**
  depend on AiOSCanvas.
- **AiOSFSS — P3 onward (transparent).** Syncs the ciphertext store; imposes
  little on AiOSComms beyond keeping runtime state outside the synced tree.
- **AiOSOrg — downstream consumer, P6.** AiOSComms exposes the interface; the
  schema is frozen for `1.0` once AiOSOrg's needs are known. AiOSComms does not
  depend on AiOSOrg, but AiOSOrg depends on AiOSComms.
- **Parent F-INTEROP — P6.** Rich in-canvas content rendering reuses the
  sandboxed-parser path rather than parsing untrusted documents unsandboxed.

## Change Log

| Date | Change | Rationale |
|------|--------|-----------|
| 2026-05-22 | Initial roadmap created | Repository kickoff; phases aligned to the parent AiOS milestones M1/M2/M3 |
| 2026-05-22 | Sequencing locked to the integration matrix — email core (Gmail + IMAP/SMTP) first and dogfoodable headless (P1), consumer interface + agent seam (P2), unified pane + Telegram at M2 (P4), sanctioned-provider breadth toward M3 (P5), AiOSOrg-facing freeze (P6) | Realistic ordering: build the legally/technically sound providers first; defer Yellow; never build Red; AiOSComms is post-M1 and downstream of vault/agent/canvas |
