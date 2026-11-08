# agent-costco-groceries

A [Claude Code](https://claude.com/claude-code) skill that fills your [Costco Same-Day](https://sameday.costco.ca) cart (Instacart, Canada) from a grocery list. **It never checks out**: you review the cart and pay yourself.

## Why

We order from Costco Same-Day every week, and the cart looks almost the same each time. On weekends we look for recipe ideas, write the grocery list, then spend time searching for the same products again and again.

This skill takes that list and fills the cart for me: it remembers which product I buy for each item, handles out-of-stock items and flags deals on what I usually buy. I only review the cart and pay, from my computer or my phone.

## What it does

- Applies your product preferences (`preferences.md`) and asks you to choose when it does not know an item. Your choice is then saved.
- If a product is out of stock, it uses the planned substitute, otherwise it suggests alternatives.
- Checks the weekly deals for your usual products, without adding them to the cart.
- Compares prices with your previous orders and archives a summary in `history/`.

Guardrails: no payment, no login or card entry, confirmation above 5 units of an item or a $200 subtotal.

## Requirements

- Claude Code
- Node.js (the `@playwright/mcp` MCP server is started with `npx`)

## Setup

```bash
git clone https://github.com/DrakkarStorm/agent-costco-groceries.git
cd agent-costco-groceries
cp config.example.md config.local.md          # delivery address and language
cp preferences.example.md preferences.md      # your usual products
cp list.example.md list.md                    # default list (optional)
```

Run `claude` in the folder and approve the `costco-browser` MCP server defined in `.mcp.json`. It uses a persistent browser profile (`.browser-profile/`): log in to your Costco account once in that window to find the cart on your other devices.

## Usage

In Claude Code:

```
/costco-groceries eggs, 2 milk, bananas, toilet paper
```

Without a list, the skill reads the "Items" section of `list.md`.

## Personal data

`config.local.md`, `preferences.md`, `list.md`, `history/`, the browser profile and the Playwright captures are ignored by git. Do not remove them from `.gitignore`: the profile holds your Costco session.
