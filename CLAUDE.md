# CLAUDE.md — AiOSComms

**Status: planning / not started.** No code exists yet; this repo is a placeholder for a future AiOS widget.

## What this is
AiOSComms is a planned **canvas widget** for AiOS — an AI-first, Linux-based OS whose desktop is a free-form canvas of widgets rendered by AiOSCanvas (a Smithay/Wayland compositor). Parent project: https://github.com/jduncan8142/AiOS .

AiOSComms is the **communication** widget: advanced email and chat unified into a single pane of glass on one canvas surface. Email starts with Gmail and local IMAP, with other providers and protocols to follow. Messaging spans SMS/text plus chat platforms — Telegram, WhatsApp, Signal, Discord, and more. The integration surface is broad and varied, which makes this expected to be among the most complex AiOS widgets.

## Scope (from the AiOS PARKING_LOT — "Planned Applications & Widgets")
- Unified communication surface: advanced email + chat in a single pane of glass (one canvas widget).
- Email: Gmail and local IMAP first; additional providers/protocols afterward.
- Messaging: SMS/text plus chat platforms — Telegram, WhatsApp, Signal, Discord, and more.
- The integration list is long and varied; **this widget needs a dedicated, clearer scoping pass before any build starts.**

## Conventions inherited from AiOS
- Integrates as a canvas widget hosted by AiOSCanvas; authoritative cross-project planning docs live in the parent AiOS repo.
- License: Apache-2.0 (AiOS-wide).
- Language/stack: **TBD at kickoff**.
- Security posture: all external/inbound content (emails, messages) is untrusted per AiOS F-PROMPT-SAFETY; design accordingly later.

## Next steps (when we start)
- Decide stack + architecture; write PLAN/ROADMAP; scaffold; wire in as an AiOS component. For AiOSComms, do a scoping pass on the integration matrix first.
