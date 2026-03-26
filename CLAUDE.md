# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file HTML application (qq-session-tool.html) for QQ temporary session management and bulk messaging. Features include:
- Single QQ session link generation
- Batch QQ session link generation
- Broadcast messaging with configurable intervals
- Operation history tracking (localStorage)

## Development

Open the HTML file directly in a browser:
```bash
start qq-session-tool.html    # Windows
open qq-session-tool.html     # macOS
```

No build step or dependencies required - it's a self-contained HTML file with embedded CSS and JavaScript.

## Architecture

**Single-file structure** - All code is in `qq-session-tool.html`:
- **CSS**: Embedded in `<style>` tag (lines 8-270). Uses CSS variables for theming with a dark theme design system.
- **JavaScript**: Embedded in `<script>` tag (lines 387-599). No external frameworks, vanilla JS only.
- **HTML**: Tab-based UI with 4 panels: single session, batch session, broadcast, history.

**Key Functions**:
- `makeURL(qq)` - Generates QQ temporary session link using `tencent://message/?uin=...` protocol
- `switchTab(name)` - Tab navigation controller
- `startBroadcast()` / `sendNext()` / `stopBroadcast()` - Broadcast messaging workflow with interval control
- History stored in localStorage under `qq_history` key (max 200 entries)

**QQ Number Validation**: 5-12 digit numbers via `isValidQQ()` regex `/^\d{5,12}$/`

## Design System

CSS variables defined in `:root` (lines 11-30):
- Colors: `--bg`, `--bg2`, `--bg3` (background layers), `--accent` (#4e9eff blue), `--text`, `--text2`, `--text3`
- Status colors: `--success` (green), `--danger` (red), `--warn` (orange)
- Typography: Noto Sans SC (UI), JetBrains Mono (code/numbers)