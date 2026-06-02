# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Portuguese flashcard app with no build step, no dependencies, and no server required. Open `index.html` directly in a browser. Both files must stay in the same folder — the HTML loads the vocabulary file via `<script src="vokabeln.js">`, which works over `file://` (unlike `fetch()`). The app is also published via GitHub Pages for iPhone use (served files: `index.html`, `vokabeln.js`, `manifest.json`, `icon.png`).

## File Structure

| File | Content |
|---|---|
| `index.html` | UI, CSS, app logic |
| `vokabeln.js` | All vocabulary (`rawCards`) + category list (`CATEGORIES`) |
| `Mein Deck.xml` | Original Anki-export — reference only, not loaded at runtime |
| `Neue Karten Portugiesisch PT.xml` | Additional vocab candidates — reference only, not loaded at runtime |

**To add or edit vocabulary: only touch `vokabeln.js`.** The HTML never needs to change for vocabulary updates.

## Architecture

### `vokabeln.js`
Exports two globals consumed by the HTML:
- `rawCards` — flat array of `[german, portuguese, category]` triples. The Portuguese string may contain `\n`-separated conjugations (line 0 = base form, lines 1–5 = eu/tu/ele/nós/eles).
- `CATEGORIES` — ordered array of category name strings, starting with `'Alle'`. Controls the order of filter buttons in the UI.

### `index.html`
- **Modes** (`mode`): `de-pt` (German front), `pt-de` (Portuguese front), `mixed` (randomly per card). Cycled via the mode button.
- **Sort order** (`sortMode`): `random` (shuffled) or `box` (sorted by Leitner box, lowest first).
- **State**: `cards[]` (filtered/sorted subset of `rawCards`), `idx`, `isFlipped`, `stats` (per-session counters), `progress` (persistent, see below).
- **`parseCard(raw)`**: splits the Portuguese string on `\n` — index 0 is the main word, the rest are conjugations shown as `·`-joined on the card back.
- **`applyFilters()`**: rebuilds `cards[]` from `rawCards` based on the active category. Called when category, mode or sort order changes.
- **`rate('good'|'bad')`**: updates per-card Leitner progress (`good` → box +1 up to 5, `bad` → box reset to 1), bumps session stats, persists, and advances.

### Progress persistence (Leitner system)
Progress **is** persisted — this is a spaced-repetition-style Leitner box system, not just session counters.
- Stored in `localStorage` under key `vokabel-progress-v1`, as a map of `cardKey → { box, good, bad, seen, last }`.
- **`cardKey(raw)` = `german + "|" + portuguese`** — i.e. the German term plus the *entire* Portuguese string (including the `\n`-conjugations). The category is NOT part of the key.
- **⚠ Editing a card's German term or Portuguese string changes its key**, so the existing progress is orphaned and the card restarts at box 1. Adding conjugations to an existing verb therefore resets that verb's progress. Adding brand-new cards, reordering, or changing only the category does NOT affect existing progress.
- `localStorage` is per-browser/per-device — desktop and the GitHub-Pages iPhone version each keep separate progress. Pushing to GitHub never touches a user's stored progress.
- Export/Import buttons save/load the progress JSON, but import merges by `cardKey`, so it cannot re-map progress after a card's text changed.

## Adding New Vocabulary

Edit `vokabeln.js` only:
1. Add each card as `["German term", "português\nconjugation1\n...", "Kategorie"]`.
2. Use an existing category from `CATEGORIES`, or add a new string to both `CATEGORIES` and the new cards.
3. Check for duplicates by searching for the German term in the file.

### Content Rules (always apply)

- **Only European Portuguese (pt-PT). Never Brazilian Portuguese.** Use pt-PT vocabulary, spelling and grammar throughout (e.g. `comboio` not `trem`, `casa de banho` not `banheiro`, `pequeno-almoço` not `café da manhã`, `telemóvel` not `celular`). If a term differs between the variants, pick the Portugal form and add a `⚠`-note warning against the Brazilian one, as seen in existing cards.
- **Verbs — two cards per verb:**
  1. In category `Verben`: base form + all 5 present-tense conjugations, `\n`-separated in the order **eu/tu/ele/nós/eles** (e.g. `"falar\nfalo\nfalas\nfala\nfalamos\nfalam"`). Do this **even for regular verbs** — always give the full set.
  2. In category `Präteritum`: the same verb with all 5 simple-past conjugations. Put the eu-form on line 0 and the full set in parentheses on line 1, matching the existing pattern (e.g. `"falei\n(falei, falaste, falou, falámos, falaram)"`).
- **Adjectives & nouns with masculine/feminine forms:** always give both, separated with `/` (masculine first, then the feminine ending), e.g. `magro/a`, `bonito/a`.
- **Nouns:** always include the article (`o`/`a`, plural `os`/`as`), e.g. `o carro`, `a mesa`, `as calças`.

Card format from the XML sources:
- `<text name="Text">` = German side
- `<text name="Übersetzung">` = Portuguese side (literal newlines become `\n`)
- XML comment headers like `<!-- VERBEN: ... -->` indicate which category to assign.

## Keyboard Shortcuts (in-app)

| Key | Action |
|-----|--------|
| Space / ↑ | Flip card |
| ← → | Previous / Next card |
| G | Rate "Gewusst" (known) |
| N | Rate "Nochmal" (again) |
