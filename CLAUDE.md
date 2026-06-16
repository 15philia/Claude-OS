# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a personal project repository ("Claude OS") for tools, games, and experiments built with Claude Code. Each project is typically a self-contained HTML file with embedded CSS and JS.

## Running Projects

All projects are standalone HTML files — open directly in a browser:
```
open tictactoe.html
```

No build step, no package manager, no server required.

## Architecture

### tictactoe.html

Single-file architecture: HTML structure, CSS styles, and JS logic are all inline.

**Game logic:**
- `board` — flat array of 9 elements (`null | 'X' | 'O'`)
- `WINS` — hardcoded array of all 8 winning index triplets
- `checkWin()` — iterates `WINS` after every move; also detects tie when board is full
- `handleClick(i)` — entry point for human moves; triggers `aiMove()` via `setTimeout(300)` when in CPU mode
- `bestMove()` / `minimax()` — unbeatable minimax AI; O maximizes, X minimizes (scores: `±(10 - depth)`)

**Score tracking:** persists across rounds within a session; resets when the mode toggle (2P ↔ CPU) is switched.

## Git & GitHub

All changes should be committed with clean messages and pushed to `https://github.com/15philia/Claude-OS`. Commit after each meaningful change; don't batch unrelated work.
