# Shopping preferences

This file is read by the `costco-groceries` skill at every session. You can edit it by hand.
The agent adds an entry to "Products" itself each time you pick a product it did not know.

## General rules

Applied in this order when several products match:

1. Prefer the Kirkland Signature brand when it exists.
2. At equivalent quality, pick the best unit price (per kg, per L or per unit).
3. Avoid "party" or very large formats, unless stated otherwise in "Products".

## Products

Entry format:

```
### <generic term, lowercase>
- Product: <exact name as shown on the site, in the configured language>
- Size: <size or packaging>
- Default quantity: <number>
- Substitute: <optional, exact name of the product to add without asking if the usual one is unavailable>
- Notes: <optional, for example "accept another brand if Kirkland is unavailable">
```

For an item you want to pick at every order, replace the Product and Size lines with `- Always ask: yes`.

<!-- Entries added by the agent appear below this line -->

### milk
- Product: Natrel 2% Milk, 2 L
- Size: 2 L
- Default quantity: 2

### toilet paper
- Product: Kirkland Signature Bathroom Tissue, 30 rolls
- Size: 30 ct
- Default quantity: 1
- Substitute: Charmin Ultra Soft Bathroom Tissue Jumbo Rolls, 30 x 200 sheets

### apples
- Always ask: yes
- Default quantity: 1
- Notes: pick the variety at every order
