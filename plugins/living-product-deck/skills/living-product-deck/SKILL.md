---
name: living-product-deck
description: "Build or EDIT continuously-growing master documentation decks for products and platforms \u2014 single HTML file with TOC + section anchors, vertical-scroll slides, populated from actual codebase state. Use when asked for a state-of-platform deck, product vision doc, internal outlook, partner brief, or any deck that consolidates multiple sub-topics into one expandable document the user will keep adding to. Also use for renumbering and structural edits (delete slides, update phase status, swap vendor mentions, refresh from real codebase state)."
version: 1.0.0
license: MIT
author: "the maintainers"
---
# Master Product Deck

A single-document deck that consolidates everything known about a product — platform state, partner briefs, internal outlooks, IB plans, build queues, revenue models — into one scrollable HTML file with a TOC and section anchors. Designed to be appended to over time, not replaced.

## When to use this pattern

Use when the user wants:

- A "state of the platform" or "where we are today" deck — sourced from the actual codebase, not memory
- A "vision" or "outlook" document they will keep adding slides to
- A consolidation of multiple sub-decks (e.g. a partner brief + an internal outlook + a build queue) into one TOC'd document
- A document they will share with a partner or investor that needs to be white-label clean and reflect ground truth

Do NOT use when the deck is a one-shot marketing piece with no plan to extend, or when the content is purely speculative (no code to audit).

### Trigger phrases that load this skill

Voice cues that should make you reach for this skill before reading any file:

- "make changes to the vision / master deck / state-of-platform doc"
- "update the deck" / "edit the slide deck" / "fix the deck HTML"
- "refresh the doc from the codebase" / "what's actually shipped vs. aspirational"
- "I already answered those questions" / "delete the open questions slide"
- "swap X for Y in the deck" / "remove all mentions of Z" / "we're no longer using V"
- "just number it correctly" / "renumber after the deletions" / "drop section X"
- "push to gh pages" / "publish the deck update" / "the org-vision.html → live"

## Structure

1. **TOC slide** (`<section class="slide toc-section" id="s0">`) — Card grid linking to each section. Update when adding sections.
2. **Section blocks** — Each is a chunk of vertical-scroll slides using `<section class="slide" id="sN">...</section>`. Sections are separated by section-break markers in the nav-rail (`<span class="section-break"></span>`).
3. **Shared design system** — One `<style>` block at the top defines tokens (colors, panels, grids, pills, checklists). All slides reuse the same tokens. ~150KB total is normal for 40+ slides.
4. **Nav rail on the right** — One `<a href="#sN">` per slide. Active state via IntersectionObserver.
5. **Deck meta top-left** — `<PRODUCT NAME> · MASTER DECK · <DATE>`.

## Sourcing content (CRITICAL)

**Never build a platform-state deck from session memory, stale docs, or aspiration.** Always:

1. **Audit the actual repo before writing:**
   - `find ./src/app -name "page.tsx"` → real dashboard routes
   - `find ./supabase/migrations -name "*.sql"` → real schema, RPCs, views
   - `grep -E "name|unlockLevel|price" src/data/*.ts` → real product lists and pricing
   - `cat src/components/<X>View.tsx` → real reward ladder / rank values (e.g. `GuideView.tsx` for OS levels)
   - `git log --oneline -30` → real recent milestones for the build timeline

2. **Cross-reference the user's stated reality.** If they say "we removed X" or "X is now $Y" — verify in the repo before quoting. Real values outrank remembered values outrank old docs.

3. **Calibrate commentary to actual values.** Don't manufacture curve logic. If the source shows $5 at L1 and $500 at L20, use those exactly and explain the calibration factually.

## White-label rule (extends the broader principle)

In ANY product/platform material — public, partner-facing, internal admin views, even state-of-platform decks — Zora/Hermes/MiniMax/Zora Cloud must NEVER appear by name:

- ❌ Never say "your + Zora's view" or "Zora reviewed this" or "Zora Cloud bot"
- ❌ Never reference agent infrastructure in product material
- ✅ Frame internal panels as "the principal + partner's view", "the team", "the platform itself"
- ✅ Even "Admin & Ops" attribution goes to the humans, not the agent
- ✅ Strip vendor mentions (Resend, etc.) from public-facing decks unless explicitly asked

This rule is broader than customer-facing. Internal platform-state decks must also be white-label clean.

## File naming

- File: `<product>-vision.html` or `<product>-master-deck.html` — short, product-name-first
- Browser tab title: same name
- Top-left meta: `<PRODUCT NAME> · MASTER DECK · <DATE>`

## How to grow the document

When the user asks to add content:

1. Append new `<section class="slide" id="sN">...</section>` block at the end (use next sequential id)
2. Add `<a href="#sN">` to the nav-rail with a `<span class="section-break"></span>` if starting a new section
3. Add a TOC card to the TOC slide if it's a new section
4. Update the closing-slide's "Continue to Section X" link to point to the new section
5. **Do NOT renumber existing slides** — preserve stable anchor IDs so external links keep working

## Pitfalls

- **Don't trust docs over code.** Master docs like the org.md can be 4+ months stale. The migrations, src/, and package.json always tell the current truth. Always audit the repo, not the docs.
- **Don't apologize for things you didn't do.** When the user says "you deleted X" but X was never in your workspace this session, push back honestly with filesystem evidence (`find`, `git log`) rather than fabricating a restoration or apologizing for something you didn't do.
- **Generic `replace_all` patches corrupt matching content across slides.** Lead paragraphs, status pills, and panel headers often share identical strings across slides. If you `replace_all` on a generic string, you wipe out the unique copy on each. Always anchor slide-specific edits with surrounding context (the slide title that follows, a unique list item, etc.).
- **Numbered lead paragraphs look identical across slides.** A bulk replace of `p class="lead"` will clobber every unique lead. Anchor each one with the slide's preceding title.
- **Inventing values breaks trust.** If the source shows $5 at L1 and $500 at L20, use those exactly. Don't round, don't "improve", don't manufacture a curve.
- **Generic "expandable" promises need real structure.** If you tell the user "we can keep adding to it," the doc must have a TOC + nav-rail + section anchors so additions actually work. Don't promise growth without scaffolding.
- **Don't mention internal vendor tools (Resend, Zora, etc.) in platform decks** unless the user explicitly says to. The white-label rule applies broadly.

### Vendor swap on an existing deck

When the user says "we no longer use X, focus on Y" or "remove all mentions of Z", do NOT just sed the bare vendor name across the file. The same vendor name lives in titles, lead paragraphs, tags, repo paths inside `<code>` tags, file paths inside ASCII code blocks, slide-cover subtitles, architecture stack rows, nav rail tooltips, and docstrings. A literal sed catches the obvious hits and leaves orphans everywhere else.

The correct pattern: pre-flight `rg -n` to list every hit, categorize each as hard-replace / verb-rephrase / compound-path / inside-code-block, run targeted `patch` calls in parallel for each category (each anchored with surrounding HTML context — never `replace_all` on the bare vendor name), then verify with `rg -c 'old|vendor|form'` returning **0** and `rg -c 'new|vendor'` matching the original hit count. Then ask whether the standalone infrastructure row should be deleted entirely or kept as legacy context. Worked example from the Jul 29 2026 the org vision edit (MT5→the broker across 17 hits in 6 contexts): `references/vendor-swap-worked-example.md`.

### Visual verification of edits — hash navigation does NOT scroll in headless browser

`browser_navigate(url='file:///path/to/file.html#s37')` returns a snapshot of the full page but does NOT scroll to the anchor. The `id="s37"` exists but the viewport stays at the top, so `browser_vision` returns an empty/blank screenshot. The notes-panel observer (slide-num display) also stays at "TOC".

To verify a specific slide visually:
- **Click the nav rail link** for the slide (e.g. `browser_click(ref=e53)` for `Hosting / Infra`). Hash scrolling in the SPA happens via the click handler, not the URL fragment.
- Or use `open_preview(url='/Users/.../file.html#s37')` — the in-app preview pane handles anchors.
- `browser_press(key='End')` works as a brute-force scroll to the last slide but doesn't tell you WHICH slide you're verifying.

If the vision screenshot returns all-black or empty, the viewport scrolled to a section-break empty band, not to a slide. Try the nav-rail click first; it deterministically scrolls to the slide the anchor points to.

## Editing existing decks — renumbering and deletions

Deletions cascade through 5+ places and a renumbering "X / N" pattern. Touch them in this order so you don't paint yourself into a corner.

### 1. Locate the exact block before you delete it

Before any `patch` that removes a `<section class="slide" id="sN">` block, read at least the line that follows it. Comment markers like `<!-- ───────────── SLIDE N+1: ... -->` sit between blocks, and the section right after a deleted block often reuses the same first-line text. If your patch's diff shows you touched the comment BELOW your target, you targeted the wrong boundary — revert immediately and re-read.

**Real mistake pattern (Jul 29 2026, the org vision deck):** I went to delete the "Dev Center" section by replacing its comment + opening `<section>`. The diff came back clean but the change actually landed on the wrong sibling block (the Build Queue comment immediately after). The fix was a one-line `read_file` to confirm the boundary, then a second patch to revert. Lesson: **a diff that looks "clean but wrong" is the most dangerous kind of patch success** — the tool's fuzzy matching can find a similar-looking match if your `old_string` isn't strictly unique.

### 2. After deletions, renumber the surviving slides in this order

When the user asks to "just number it correctly" (or wants slides deleted with renumbering), touch these in order. Each step's grep pre-flight proves the previous step landed cleanly:

1. **Section IDs in HTML** — `id="sN"` → `id="sN-1"` for the slide that took the deleted slide's number. Read first; unique.
2. **slide-num strings** — `<div class="slide-num">N / 42</div>` for both the renumbered slide AND the new total. Use `sed -i '' 's|<div class="slide-num">\([0-9][0-9]*\) / OLD_TOTAL</div>|<div class="slide-num">\1 / NEW_TOTAL</div>|g' file.html` to swap the total once across all surviving slides.
3. **Comment-block markers between slides** — `<!-- ───────────── SLIDE N: ... -->` headers above each slide need their number updated (and the title text updated if the section was retitled by the deletion cascade). Grep `SLIDE [0-9]` and `id="sN"` to confirm parity.
4. **Nav rail anchors** — `<a href="#sN">` entries for the deleted slides. Read the nav-rail block; it's a tight list and easy to patch as one chunk.
5. **Closing-slide back-link** — most decks end with a `<a href="#s0">← Back to TOC</a>`. Confirm `#s0` is still correct (it usually is, but if you also deleted/rebuilt the TOC, the anchor shifts).
6. **TOC card counts** — `SLIDES · 14` / `(14 slides)` references in the TOC card meta and in the section header comment. Grep for both forms before patching.
7. **Section cover paragraph** — the cover slide for that section often says "14-slide deep dive" / "14 slides total" inline. Grep for the literal phrase.

### 3. Pre-flight verification grep pattern

After deletions + renumbering, run these four greps in one terminal command:

```bash
grep -oE 'id="s[0-9]+"' file.html | sort -u | wc -l   # unique slide IDs
grep -cE 'slide-num">[0-9]+ / NEW_TOTAL' file.html    # all display strings match new total
grep -cE 'slide-num">[0-9]+ / OLD_TOTAL' file.html    # must be 0
grep -oE 'href="#s[0-9]+"' file.html | sort -u       # nav anchors match slide IDs
```

If `id="sN"` count != nav-anchor count, the deck has dead nav links. If `OLD_TOTAL` count is non-zero, a slide-num string slipped through the sed.

### 4. The `<section class="slide">` grep-count trap

Counting `<section class="slide"` and `</section>` to verify the file is balanced will show off-by-one — the file's **own docstring** near the top usually contains the literal string `<section class="slide">` as an example, and `grep -c` matches it. That doesn't count toward the slide count. Fix: use `grep -oE 'id="s[0-9]+"' file.html | sort -u` instead, which counts actual slide elements by their IDs.

### 5. Status-pill additions when content updates with reality

When a phase/status card on a slide changes from "planned" to "actually in progress", don't just update the text — add a visible status strip with a `.pill` and a one-line reality check. The user reads these decks to ground themselves, and "what we said vs. what happened" is the whole point. Pattern:

```html
<div style="margin-top:14px;padding-top:14px;border-top:1px solid var(--border);">
  <div style="font-family:var(--font-mono);font-size:10px;color:var(--text-muted);
              text-transform:uppercase;letter-spacing:0.08em;margin-bottom:8px;">
    Status — Jul 29
  </div>
  <div style="display:flex;gap:8px;flex-wrap:wrap;">
    <span class="pill wip">WIP</span>
    <span style="font-size:12px;color:var(--text-secondary);">
      Scanner import not started — task seeded, due Aug 10
    </span>
  </div>
</div>
```

Date-stamp the status line so the user knows which session's reality it reflects.

### 6. Pull prior-session decisions from session_search before asking the user

If a deck contains "open decisions" / "pending" / "PENDING" sections that the user said they've already answered, don't ask them to repeat the answers — `session_search` for the prior session where they locked the calls. Typical pattern:

```bash
session_search(query="<exact decision text from the slide>", limit=3)
```

Then `session_search(session_id="...", around_message_id=<the answer message>)` to grab the verbatim call. The user gets annoyed when you ask them to repeat decisions they already made — that "yes I already said this" friction is the whole reason session memory exists. Apply this whenever the user says "update this from yesterday" or "I already answered those" in a session that has prior context.

## Output format

Single self-contained HTML file, vertical scroll-snap, dark theme, nav-rail on the right, TOC landing at top. Open with `open_preview` for visual verification. The file should be openable in any browser without a build step.

## Related skills

- `static-landing-pages` — for marketing pages, not internal documentation
- `visual-plan-deck` — for one-shot plan decks; this skill is for product master docs that grow over time
- `cross-repo-tech-showcase` — for cross-repo knowledge bases (different format)