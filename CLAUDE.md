# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file HTML application (qq-session-tool.html) for QQ temporary session management and bulk messaging. Features include:
- Single/batch QQ session link opening
- Batch add friend functionality
- Configurable batch size and popup auto-close timing
- Theme toggle (dark/light mode)
- Settings persistence via localStorage

## Development

Open the HTML file directly in a browser:
```bash
start qq-session-tool.html    # Windows
open qq-session-tool.html     # macOS
```

No build step or dependencies required - it's a self-contained HTML file with embedded CSS and JavaScript.

## Platform Limitations (Critical)

**QQ temporary sessions have inherent restrictions:**
- `wpa.qq.com` can only **open chat windows**, cannot auto-send messages
- No web protocol exists for auto-sending to strangers
- QQ Bot API only works for friends/channel members
- Third-party protocols (OneBot) require shared group or friend relationship

**Browser popup limitations:**
- Firefox blocks multiple `window.open` calls without delay
- Chrome/Edge may also limit concurrent popups
- Solution: Add interval delay between window opens

## Architecture

**Single-file structure** - All code in `qq-session-tool.html`:
- **CSS** (lines 10-669): CSS variables theme system, dark/light modes
- **JavaScript** (lines 770-1021): Vanilla JS, no frameworks
- **HTML**: Single panel with broadcast controls, confirmation modal

**Key Functions**:
- `makeURL(qq, mode)` - Generates link: `wpa.qq.com` (session) or `wp.qq.com` (add friend)
- `startBroadcast(mode)` - Parses QQ list, shows confirmation modal
- `openBatch(count)` - Opens `window.open` calls with position offset
- `closeAllOpenedWindows()` - Closes tracked window references

**State Management**:
- `bcQueue`, `bcIndex` - Queue and progress tracking
- `bcBatchSize`, `bcStayTime` - Settings from localStorage
- `openedWindows[]` - Window references for cleanup

**QQ Validation**: `/^\d{5,12}$/` via `isValidQQ()`

## Design System

CSS variables in `:root`:
- Background layers: `--bg`, `--bg2`, `--bg3`, `--bg4`
- Primary: `--primary` (#3b82f6 blue), hover states, glow effects
- Text: `--text`, `--text2`, `--text3` (opacity hierarchy)
- Semantic: `--success`, `--danger`, `--warn` with backgrounds
- Fonts: Inter (UI), JetBrains Mono (numbers/mono)