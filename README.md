# Living Product Deck: Docs That Update Themselves

> A single self-updating HTML product document: table of contents, anchored sections, vertical-scroll slides, populated from the actual codebase so the deck stays true as the product moves.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg) ![Claude Code](https://img.shields.io/badge/Claude_Code-skill-blueviolet.svg) [![Available on Agensi](https://img.shields.io/badge/Agensi-%2414.99-ff5a5f.svg)](https://agensi.io/skills/living-product-deck-docs-that-update-themselves)

## What it does

Build or EDIT continuously-growing master documentation decks for products and platforms — single HTML file with TOC + section anchors, vertical-scroll slides, populated from actual codebase state. Use when asked for a state-of-platform deck, product vision doc, internal outlook, partner brief, or any deck that consolidates multiple sub-topics into one expandable document the user will keep adding to. Also use for renumbering and structural edits (delete slides, update phase status, swap vendor mentions, refresh from real codebase state).

A single-document deck that consolidates everything known about a product — platform state, partner briefs, internal outlooks, IB plans, build queues, revenue models — into one scrollable HTML file with a TOC and section anchors. Designed to be appended to over time, not replaced.

## The failure it prevents

The cases that cost time are specific, and they are written down:

- **Don't trust docs over code.** Master docs like the org.md can be 4+ months stale. The migrations, src/, and package.json always tell the current truth. Always audit the repo, not the docs.
- **Don't apologize for things you didn't do.** When the user says "you deleted X" but X was never in your workspace this session, push back honestly with filesystem evidence (`find`, `git log`) rather than fabricating a restoration or apologizing for something you didn't do.
- **Generic `replace_all` patches corrupt matching content across slides.** Lead paragraphs, status pills, and panel headers often share identical strings across slides. If you `replace_all` on a generic string, you wipe out the unique copy on each. Always anchor slide-specific edits with surrounding context (the slide title that follows, a unique list item, etc.).
- **Numbered lead paragraphs look identical across slides.** A bulk replace of `p class="lead"` will clobber every unique lead. Anchor each one with the slide's preceding title.
- **Inventing values breaks trust.** If the source shows $5 at L1 and $500 at L20, use those exactly. Don't round, don't "improve", don't manufacture a curve.
- **Generic "expandable" promises need real structure.** If you tell the user "we can keep adding to it," the doc must have a TOC + nav-rail + section anchors so additions actually work. Don't promise growth without scaffolding.

## When to use it

- build a product documentation deck
- keep a deck in sync with the product
- turn repo state into a stakeholder update

## What's inside

```
plugins/
└── living-product-deck/
    ├── .claude-plugin/plugin.json
    └── skills/living-product-deck/
        ├── SKILL.md
        ├── references/            (1 files)
        │   └── vendor-swap-worked-example.md
```

## Install

Add this repo as a Claude Code marketplace, then install the skill:

```bash
claude plugin marketplace add Verstige/living-product-deck
claude plugin install living-product-deck@living-product-deck
```

Or copy the skill straight into your skills directory:

```bash
git clone https://github.com/Verstige/living-product-deck.git
cp -R living-product-deck/plugins/living-product-deck/skills/living-product-deck ~/.claude/skills/
```

## Usage

The skill activates on the work it describes. Ask for the outcome, and it
supplies the method:

```
> "build a product documentation deck"
```

## Reference material

Loaded on demand — the skill only reads what the task needs.

| File | Covers |
| --- | --- |
| `vendor-swap-worked-example.md` | Vendor Swap on a Master Deck — Worked Example |

## License

MIT. See [LICENSE](LICENSE).

---

**Also available as a paid skill on Agensi** — $14.99. Install it with one command, no clone required:

[agensi.io/skills/living-product-deck-docs-that-update-themselves](https://agensi.io/skills/living-product-deck-docs-that-update-themselves)

Part of the [Verstige](https://github.com/Verstige) skills portfolio. More at [github.com/Verstige](https://github.com/Verstige).
