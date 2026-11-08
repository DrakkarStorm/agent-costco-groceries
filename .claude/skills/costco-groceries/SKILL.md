---
name: costco-groceries
description: Fills the Costco Same-Day cart (sameday.costco.ca) from a grocery list, applying the preferences in preferences.md, without ever checking out. Use this skill when the user asks to do their Costco groceries, fill their Costco cart, or gives a grocery list for Costco.
---

# Costco Same-Day groceries

You fill the cart at https://sameday.costco.ca/store (Instacart platform) from a list. The user pays, never you.

Project files (at the repository root):
- `config.local.md`: delivery address and language
- `preferences.md`: general rules and known products
- `list.md`: default list
- `history/YYYY-MM-DD.md`: archived summaries

These files are personal and ignored by git. If `config.local.md`, `preferences.md` or `list.md` does not exist, create it by copying the matching template (`config.example.md`, `preferences.example.md`, `list.example.md`). If `config.local.md` still holds the template values, ask the user for their address, postal code and language, and save them. Create the `history/` folder if it is missing.

## Language

Read `Language` in `config.local.md`. Talk to the user, write the summary and save new preferences in that language. The site must use that language too: if the language selector in the header shows another one, switch it before searching for anything. Product names saved in `preferences.md` and `history/` are in that language.

## Browser

Use **only** the tools of the `costco-browser` MCP server (`mcp__costco-browser__browser_*`). Its profile is persistent, which lets the user log in in the same window at the end. Do not use the global Playwright server: its profile is different and the cart would be lost.

To read the page, use `browser_snapshot` rather than screenshots. Selectors may change: always locate elements in the snapshot, never assume where they are.

Browse like a human: one action at a time, no parallel requests, no catalog exploration beyond the items on the list and the weekly savings page (step 5).

## Guardrails (non-negotiable)

- Never click "Checkout", "Place order", "Pay" or any equivalent button.
- Never enter a username, password, payment card or membership number. If the site requires one to continue, stop and explain the situation to the user.
- Before adding a quantity greater than 5 for an item, ask for confirmation.
- If the estimated subtotal exceeds $200, stop adding items and ask for confirmation before continuing.
- Do not close the browser at the end: the user must be able to log in and pay.

## Steps

### 1. Load the list and preferences

- If the user gave a list in the message, use it. Otherwise, read the "Items" section of `list.md`. If it is empty, ask for the list.
- Read `preferences.md` in full.
- For each line, extract the quantity (if given) and the generic term.

### 2. Open the site as a guest

1. Go to `https://sameday.costco.ca/store`.
2. If the home page offers to log in via Costco.ca or to browse as a guest, choose **browse as a guest**. If the user is already logged in (persistent profile), continue as is.
3. If the site asks for an address or postal code, enter the postal code and address from `config.local.md` and confirm. Let the site pick the warehouse. Note the warehouse name shown, for the summary.

### 3. Empty the cart

Open the cart. If it contains items, remove them all without asking, then check that it is empty.

### 4. Add each item

For each item on the list, in order:

1. **Resolve the product**
   - If the `### <term>` entry in `preferences.md` contains `Always ask: yes`, search for the term, keep 2 or 3 candidates by applying the general rules (flag promotions) and present them with `AskUserQuestion`. Do not save the choice in `preferences.md`. The quantity follows the same rules as below.
   - Otherwise, if `preferences.md` contains a `### <term>` entry (same term, or an obvious synonym), search for that exact product on the site. The quantity is the one on the list, otherwise the entry's default quantity, otherwise 1.
   - Otherwise, search for the term with the search bar. Keep the 2 or 3 best candidates by applying the general rules in `preferences.md`. Present them to the user with `AskUserQuestion` (name, size, price, unit price if shown) and wait for the answer. Then add an entry to the "Products" section of `preferences.md`, following the format given in that file, below the comment line.
2. **Availability**: if the product is out of stock or not found, add nothing without consent and follow this order:
   - If the `preferences.md` entry has a `Substitute:` line and that product is available, add it directly: the line counts as the user's consent. Otherwise, go to the next point.
   - Search for the generic term (for example "toilet paper", not "charmin") and keep 2 or 3 candidates by applying the general rules in `preferences.md`. Present them with `AskUserQuestion` (name, size, price, unit price if shown), with a "Take nothing" option, and wait for the answer.
   - If the user declines, or if there is no candidate, mark the item as unavailable.
   - A substitute applies to this session only: do not change the `preferences.md` entry. Note it for the "Substitutions" section of the summary.
3. **Add**: add the product to the cart with the right quantity. Check in the snapshot that it was added (cart counter or quantity button updated). Note the displayed price and, if it is on sale, the discount amount.
4. **Threshold**: after each addition, look at the cart subtotal. If it exceeds $200, apply the guardrail.

### 5. Deals on usual products

Go to `https://sameday.costco.ca/store/costco-canada/collections/rc-weekly-savings` and read the page. If it loads products on scroll, scroll to the end.

- Usual products are those in the "Products" section of `preferences.md` and those in the "Added product" column of `history/`, excluding substitutes.
- Keep only the usual products that appear on this page and are not in the cart. Keep no other product, even with a big discount.
- Note the displayed price and the discount amount.
- Never add these products to the cart and do not ask: you only note them for the summary.
- If the page is unreachable, continue and say so in the "Deals on your usual products" section.

### 6. Summary

Open the cart and note the displayed subtotal.

Then compare prices with `history/`:

- For each product in the cart, find the most recent file that contains it. Older files may name it in another language, and their headings and columns may be in French ("Produit ajouté", "Prix"): same brand and same size means same product. If the product is not there, there is nothing to compare.
- The "Price" column holds the line total: compare the unit price (price divided by quantity), never the total.
- Flag any unit price change, up or down, and say whether a promotion explains it (promotion running today, or promotion ended since the last purchase).
- If a product is on sale today, flag it even if it has no history.
- If there is no change or promotion to report, write "No change" in the section. If a "Deals on your usual products", "Substitutions" or "Unavailable" section is empty, write "None".

Produce this summary in the chat, translated into the configured language:

```markdown
# Costco groceries for YYYY-MM-DD

Warehouse: <displayed name>

| Requested | Added product | Qty | Price |
|---|---|---|---|
| milk | Natrel 2% Milk 2 L | 2 | $11.98 |
| toilet paper | Charmin Ultra Soft 30 ct | 1 | $28.59 (on sale, $6.50 off) |

**Estimated subtotal**: $XX.XX (excluding delivery fees, taxes and tip)

## Price changes
- <product>: $X.XX → $Y.YY per unit (+$Z.ZZ since YYYY-MM-DD)
- <product>: on sale now, $X.XX off

## Deals on your usual products
- <product>: $X.XX ($Y.YY off), not added to the cart

## Substitutions
- <term>: <usual product> (<reason>) → <added product>

## Unavailable
- <term>: <reason: out of stock, not found, substitute declined>

## New saved preferences
- <term> → <product>
```

Then write the same content to `history/YYYY-MM-DD.md`, with today's date. If the file already exists, add a `-2`, `-3`, etc. suffix.

### 7. Handoff

End with a message to the user, depending on the situation:

- **Already logged in:** the cart is saved on their account. They can review it and pay from any device logged into the same account, such as their phone, or in the browser window.
- **As a guest:** the cart is ready in the browser window. They must log in via Costco.ca in that same window (the cart is kept after logging in), review the cart, then pay themselves.
