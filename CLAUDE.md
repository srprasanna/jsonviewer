# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a single-file, zero-build JSON viewer web app hosted on GitHub Pages at `jsonviewer.srprasanna.dev`. The entire application lives in `index.html` (~1250 lines). There is no package manager, bundler, or compilation step.

## Development

**To run locally:** Open `index.html` directly in a browser — no server needed.

**To deploy:** Push to `main`; GitHub Pages serves `index.html` automatically via the `CNAME` record.

There are no build, lint, or test commands.

## External Dependencies (CDN)

- **CodeMirror 5.65.16** — editor with JSON syntax highlighting, folding, bracket matching
- **Ajv 6.12.6** — JSON Schema draft-07 validation

All dependencies are loaded from CDN in `<head>`; there is no local `node_modules`.

## Architecture

The `index.html` file is divided into three sections:

1. **CSS** (lines ~26–493): Theme system using CSS custom properties. Light/dark mode is toggled via `data-theme` on `<html>`, persisted in `localStorage`.
2. **HTML** (lines ~495–610): Two-panel layout — left panel (CodeMirror editor + optional schema editor), right panel (tree view + search + breadcrumb). Mobile uses a tabbed stacked layout below 768px.
3. **JavaScript** (lines ~614–1253): All app logic as plain functions operating on a small set of globals.

### Key globals

| Variable | Purpose |
|---|---|
| `editor` | CodeMirror instance for JSON input |
| `schemaCM` | CodeMirror instance for JSON Schema |
| `ajvInstance` | Ajv validator instance |
| `currentData` | Parsed JSON object (source of truth for the tree) |
| `currentPath` | Breadcrumb path of the selected node |
| `selectedMain` | Currently highlighted tree node element |

### Data flow

User input (paste / file upload / URL / drag-drop) → CodeMirror editor → `parseJSON()` (700ms debounce or manual trigger) → `currentData` → `renderTree()` builds DOM recursively → user interacts with tree nodes (expand/collapse, search, select, copy).

### Key functions

- `parseJSON()` — parses editor content, updates `currentData`, calls `renderTree()`
- `renderTree(data, container)` — recursively builds tree nodes from any JSON value
- `searchTree(query)` — marks `.search-match` nodes and dims non-matches; auto-expands parents
- `validateSchema()` — runs Ajv against `currentData` using the schema editor content
- `loadFromURL(url)` — fetches JSON via network, populates editor

### Keyboard shortcuts (defined in `initKeyboardNavigation`)

| Shortcut | Action |
|---|---|
| `Ctrl/Cmd+Enter` | Parse JSON |
| `Ctrl/Cmd+Shift+F` | Format (pretty-print) JSON |
| `Arrow Up/Down` | Navigate tree nodes |
| `Arrow Left/Right` | Collapse/expand + navigate into children |
| `Space` | Toggle expand/collapse |
