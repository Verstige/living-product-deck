# Vendor Swap on a Master Deck — Worked Example

A worked example from the the org OS vision deck edit (Jul 29 2026) where the user asked to remove all MT5 references and replace them with the broker, across 17 hits spanning 6 different surface contexts.

## Why this is its own pattern

Vendor/platform swaps on a master deck are deceptively easy to half-do. The same vendor name appears in titles, lead paragraphs, tags, file paths in code blocks, repo URLs, slide-cover subtitles, nav rail tooltips, and the inline architecture diagrams. A `sed -i` on the literal string catches the obvious ones but misses context-specific phrasing ("MT5 wired" → "the broker wired" needs different verbs; "MT5 execution bridge · /server/mt5_bridge" is one logical line; `bridge.py # Telegram → MT5` lives inside a `<div class="code-block">` ASCII tree).

If you stop after the literal sed, you ship a deck that says "the broker" in the headline and "MT5" three slides later in the file structure tree. Worse, future grep-based audits will look clean.

## The pre-flight: scope the swap before you touch anything

```bash
# 1. List every literal hit with line numbers
rg -n 'MT5|mt5|MetaTrader|metatrader' file.html

# 2. Read each hit with enough context to see what KIND of reference it is
#    (title, lead, tag, repo path, code block comment, slide cover, nav tooltip)
```

Categorize each hit:
- **Hard replacement** — same noun, just swap (e.g. `MT5 Execution` → `the broker Execution`)
- **Verb rephrase** — the headline needs a different verb because the broker isn't an "MT5 wired" thing, it's "the broker wired"
- **Compound path** — repo paths like `the org/mt5-execution-service` are one logical unit; split-replace would leave orphan fragments
- **Inside code blocks** — `bridge.py # Telegram → MT5` lives in an indented ASCII tree; preserve the tree alignment

## The execution: targeted `patch` per category

For each category, do ONE `patch` per unique `old_string` (anchored with surrounding context — never `replace_all` on the bare vendor name). Run them in parallel where they don't depend on each other.

Example from this session — six patches executed in parallel for the MT5→the broker sweep:

```python
# Hard replacements (4)
patch("MT5 Execution" → "the broker Execution")
patch("telegram_signals ingestion, MT5 execution bridge" → "telegram_signals ingestion, the broker execution bridge")
patch("<span class=\"tl-tag\">MT5</span>" → "<span class=\"tl-tag\">the broker</span>")

# Verb rephrase (1) — the title uses different verbs because the noun changed
patch("Six features, <em>MT5 wired</em>" → "Six features, <em>the broker wired</em>")

# Compound path (1) — repo + service name as one logical unit
patch("the org/mt5-execution-service" → "the org/the broker-execution-service")

# Inside code block (1) — preserve ASCII tree indentation
patch("│   ├── bridge.py         # Telegram → MT5" → "│   ├── bridge.py         # Telegram → the broker")
```

Then **delete the old infrastructure row** if it's gone from the product. In this case the user said "we are no longer using MT5" — so the standalone MT5-row in the architecture stack was deleted entirely, not just renamed.

## The verification: zero hits, in EVERY form

After the swap, run the pre-flight search again. The success criterion is `total_count: 0` for every form of the old vendor name — including case variants, abbreviated forms, and references inside comments:

```bash
rg -c 'MT5|mt5|MetaTrader|metatrader|MetaTrader' file.html   # must be 0
```

Then verify the new vendor appears in the contexts you expected:

```bash
rg -c 'the broker' file.html   # should match the original hit count, not less
```

If the new count is less than the old, you missed a context. If the new count is significantly more, you double-stamped something.

## Pitfalls specific to vendor swaps

- **`replace_all` on the bare vendor name** will match strings you didn't intend (e.g. "metatrader's sister product" or a docstring example). Always anchor each patch with surrounding HTML context.
- **Code blocks have indentation** that gets stripped if you paste the `old_string` from a search output without preserving the leading whitespace. Read the file with `read_file` (which preserves spaces), not `grep` output.
- **Inside `<code>` tags** like `<code>the org/mt5-execution-service</code>`, the whole `<code>...</code>` is one unit. Don't split-replace.
- **Adjacent slide titles may share substrings.** "Six features, MT5 wired" and a future "MT5 service file structure" both contain "MT5" — but only one is on the slide you're editing. Anchor with the unique surrounding text (the slide title that follows, or a unique list item).
- **Tooltips and nav rail `title=` attributes** often paraphrase the slide title ("MT5 wired" → title="Six features"). Don't forget the nav rail sweep.
- **The architecture stack diagram** is its own thing. It lives in the stack slide as a list of `arch-layer` divs. If the vendor row is gone, **delete the whole `<div class="arch-layer">...</div>`** block — don't leave a placeholder.

## What to capture in the skill itself

This is a session-specific reference. The class-level lesson is "vendor swaps need pre-flight categorization + parallel targeted patches + zero-hit verification" — that's worth keeping in SKILL.md as a generalizable pitfall, not just buried here.