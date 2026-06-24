# Discord Developer Platform Gotchas

Recurring failure modes when relying on the Discord Developer Platform reference, and what to do instead. Read alongside `SKILL.md`.

- The Discord API evolves and this is a point-in-time capture; verify endpoints, fields, intents, gateway events, and permission flags against the current docs and the API version you target (for example v10), and check the change-log for recent changes.
- Field/parameter tables use Discord's exact notation: a trailing `?` on a field name means optional, a leading `?` on the type means nullable, and `?field?` means both - preserve these markers because they change how you build requests and parse responses.
- The docs cover three distinct surfaces: the REST + Gateway API and bots (resources, interactions, topics, events, gateway, components), embedded Activities (the Embedded App SDK), and the C/C++ Discord Social SDK (`discord-social-sdk` pages). Confirm you are using the surface that matches the task.
- Treat snowflake IDs, opcodes, intent values, permission bit values, gateway event names, OAuth2 scopes, endpoint paths, and rate-limit numbers as exact reference facts.
- Content is split across files by section/page; the docs are organized by area (resources, interactions, topics, events, activities, monetization, `discord-social-sdk`). Use `references/INDEX.md` or `rg -n "PATTERN" references/*.md` to find the exact page.
