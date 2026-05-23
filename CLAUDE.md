# CLAUDE.md — AiOSComms

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

**Status: planning.** No application code exists yet. The repository holds the
license and the planning document set ([PLAN.md](PLAN.md), [ROADMAP.md](ROADMAP.md)).
Read the docs before writing anything — they encode scope and decisions that no
source file reflects yet.

## What this is

AiOSComms is the **communication widget** for AiOS — an AI-first, Linux-based OS
whose desktop is a free-form canvas of widgets rendered by AiOSCanvas (a
Smithay/Wayland compositor). Parent project: https://github.com/jduncan8142/AiOS .

It unifies **email and chat into a single pane of glass** on one canvas surface,
over heterogeneous providers: email (Gmail + generic IMAP/SMTP first), then chat
(Telegram first, then a sanctioned subset). It is **the broadest integration
surface and the most prompt-injection-exposed widget in AiOS** — every inbound
message is attacker-controllable content.

## Planning documents

- [PLAN.md](PLAN.md) — vision, principles, architecture (headless core + thin
  frontends), the **provider integration matrix** (per-platform technical *and*
  legal/ToS feasibility), the **data model + AiOSOrg-facing consumer interface**,
  the **untrusted-content / F-PROMPT-SAFETY** subsystem, credentials via
  AiOSVault, the canvas UI, scope/non-scope, proposed stack, and open questions.
- [ROADMAP.md](ROADMAP.md) — phases P0–P7 mapped to AiOS milestones M1/M2/M3.

The **parent [AiOS](https://github.com/jduncan8142/AiOS) repo** holds the
authoritative cross-cutting docs — `PLAN.md`, `DECISIONS.md`, `FEATURES.md`,
`THREAT_MODEL.md`, `ARCHITECTURE.md`, `PARKING_LOT.md`. They are the source of
truth for licensing, the language split, the security posture, and milestones;
this repo's docs defer to them on cross-cutting matters.

## The three things that define this component

1. **Unified abstraction over heterogeneous providers.** A Gmail thread, an IMAP
   conversation, and a Telegram chat are all `Thread`s of `Message`s. The
   provider-neutral Account/Thread/Message/Contact model is the load-bearing API;
   connectors are the only code that knows a provider's quirks.
2. **All inbound content is untrusted data, never instructions.** This is the
   highest-risk widget for prompt injection in AiOS. The privileged agent planner
   never sees raw message bodies; an unprivileged reader extracts structured safe
   summaries. F-PROMPT-SAFETY is the spine of the design, not a later pass.
3. **A versioned consumer interface, because AiOSOrg depends on it.** A future
   organizational widget (parent PARKING_LOT #5) turns messages into tasks via a
   documented interface — read, subscribe, send, mark/flag, `extract_actionable`.
   It is designed and versioned before its consumer exists.

## Conventions inherited from AiOS

Decided AiOS-wide (parent `DECISIONS.md`), and they apply here:

- **License** — Apache 2.0 across all code.
- **Language** — Rust for system/security-critical components, Python for the
  agent and tooling. AiOSComms's hostile-input-parsing core is security-critical
  → **Rust headless core**; the agent/AiOSOrg client → **Python**. (Proposed in
  PLAN.md; confirm in Phase 0.)
- **Credentials in AiOSVault** — every provider credential (OAuth, app password,
  bot token, session key) lives in AiOSVault, mediated where the protocol allows.
  AiOSComms never stores a raw secret.
- **F-PROMPT-SAFETY** — command/data separation from the start.
- **Multi-user-ready** — single-user today, but every account/thread/message/
  contact carries a user scope from day one.
- **Local-first, encrypted-everything** — the store is ciphertext at rest;
  cross-device sync is AiOSFSS over the LAN, not a cloud account of AiOSComms's.

## Hard constraints from the integration scoping pass

- **WhatsApp personal accounts are out of scope** — there is no lawful
  third-party-client path (Meta ToS bans unofficial/automated clients). Do not
  build it. (A *Business* API integration would be a different product.)
- **Discord is bot-scope only** — self-bots (automating a user account) are
  banned on sight. Never automate a Discord user account.
- **Signal and SMS are deferred** — Signal for ToS/maintenance cost; SMS until a
  phone-companion app exists. Revisit, don't build early.

See the integration matrix in PLAN.md for the full per-platform reasoning.

## Next steps (when we start)

Phase 0 per ROADMAP.md: confirm the proposed stack and integration order with
Jason, scaffold the Cargo workspace, and write `DESIGN.md` (connector trait,
unified-model + store schema, the untrusted-content boundary, and the
consumer-interface schema + versioning policy) plus the component `THREAT_MODEL.md`.
