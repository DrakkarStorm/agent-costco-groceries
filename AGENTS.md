# AGENTS.md

Guidance for coding agents working on this repository.

## What this is

A Claude Code skill (`.claude/skills/costco-groceries/SKILL.md`) that fills a Costco Same-Day cart (sameday.costco.ca, Instacart) from a grocery list. There is no application code: the skill is a set of instructions, and the browser is driven through the `costco-browser` Playwright MCP server defined in `.mcp.json`.

## Layout

- `.claude/skills/costco-groceries/SKILL.md`: the skill itself. Most changes happen here.
- `.mcp.json`: the `costco-browser` MCP server, with a persistent profile in `.browser-profile/`.
- `*.example.md`: templates for the personal files.
- `README.md`: user-facing documentation.

Personal files, ignored by git: `config.local.md` (address, language), `preferences.md`, `list.md`, `history/`, `.browser-profile/` (logged-in Costco session) and `.playwright-mcp/` (page captures).

## Rules

- Never commit personal files, never remove them from `.gitignore`, and never put an address, name, email, door code or receipt token in a versioned file.
- Keep the skill's guardrails: never check out or pay, never enter credentials, card or membership numbers, confirm quantities above 5 and subtotals above $200.
- When you change the format of `preferences.md`, `list.md`, `config.local.md` or the history summary, update the matching `*.example.md` template and the README in the same change. Old `history/` files are never rewritten, so the skill must keep reading earlier formats (including French headings).
- Versioned files are written in English. The language used with the user and on the site comes from `Language` in `config.local.md`.
- There are no automated tests. To check a change to the skill, run it in Claude Code with a short list and review the cart and summary; it never places an order.
