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
- **Modes**: `all`, `de-pt` (German front), `pt-de` (Portuguese front).
- **State**: `cards[]` (filtered/shuffled subset of `rawCards`), `idx`, `isFlipped`, `stats`.
- **`parseCard(raw)`**: splits the Portuguese string on `\n` — index 0 is the main word, the rest are conjugations shown as `·`-joined on the card back.
- **`applyFilters()`**: rebuilds `cards[]` from `rawCards` based on the active category. Called when category or mode changes.
- **`rate('good'|'bad')`**: increments session stats and advances to the next card. No spaced-repetition — ratings are session-only counters.

## Adding New Vocabulary

Edit `vokabeln.js` only:
1. Add each card as `["German term", "português\nconjugation1\n...", "Kategorie"]`.
2. Use an existing category from `CATEGORIES`, or add a new string to both `CATEGORIES` and the new cards.
3. Check for duplicates by searching for the German term in the file.

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
