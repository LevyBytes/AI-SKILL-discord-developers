---
name: discord-developers
description: Use when building on the Discord developer platform: bots and apps, slash/application commands and interactions (buttons, select menus, modals, message components), the HTTP/REST API and Gateway (WebSocket) with opcodes, intents, and events, and the REST resources - application, channel, message, guild, user, webhook, emoji, sticker, invite, voice, audit-log, auto-moderation, poll, role-connection, scheduled-event, stage-instance, soundboard, SKU/entitlement/subscription. Also covers OAuth2 and bot authorization, permissions and permission bit flags, embeds, snowflake IDs, rate limits, monetization, Rich Presence, embedded Activities (the Embedded App SDK), and the C/C++ Discord Social SDK. Use this when answering how a Discord endpoint, gateway event, interaction, intent, permission, or object field works, what parameters/JSON an API call takes, or which scope/intent is required. The official Discord developer docs. Not for the end-user Discord app or general server administration.
covers:
  - design-patterns
  - local-development-mobile
  - user-actions
  - how-activities-work-overview
  - bots-overview
  - change-log
  - components-overview
  - components-reference
  - community-resources
  - communication-features
  - account-linking-on-mobile
  - account-linking-with-discord
---

# Discord Developer Platform Reference

Original-prose reference for Discord Developer Platform, split into one focused file per subject across 124 reference files.
Identifiers (API/type/function names, flags, paths, enums, numbers) are preserved verbatim.

## When to use this

Use this skill when the task involves building on Discord - bots/apps, application commands and interactions/components, the REST API and Gateway (intents, opcodes, events), the API resources (channels, messages, guilds, users, webhooks, ...), OAuth2 scopes, permissions and bit flags, embeds, rate limits, monetization, Rich Presence, Activities (Embedded App SDK), and the C/C++ Social SDK.

## Workflow

1. Open `references/INDEX.md` (or `references/topics.json`) and pick the one file that matches.
2. Read only that file; open another only if the task spans subjects. Grep across files when needed: `rg -n "PATTERN" references/*.md`.
3. Treat every identifier as an exact reference fact - never invent or paraphrase.

## Gotchas

Recurring failure modes and what to do instead live in the sibling [GOTCHA.md](GOTCHA.md).

## References

All depth lives in `references/` - start at `references/INDEX.md`, metadata in `references/topics.json`. One subject per file.

## Official documentation

The official documentation is at https://docs.discord.com/developers/intro - consult it to verify anything uncertain, conflicting, or version-specific.

## Verification

```powershell
python D:\dev\SKILLS\skills\skill-drafting\scripts\validate_skill_package.py D:\dev\SKILLS\skills\discord-developers
```
