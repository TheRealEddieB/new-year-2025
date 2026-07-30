# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo is a single self-contained static web page: a "Global New Year Countdown" app that shows live countdown timers to midnight (Jan 1, 2026) across up to 5 user-selected timezones simultaneously.

There is no build system, package manager, dependency file, test suite, or server — the entire app is one HTML file with inline `<style>` and `<script>` tags and zero external dependencies (no CDN links, no frameworks).

## Repository structure

- `Index.html` and `GlobalNYE.html` — **byte-identical duplicates** of the same app (only differs by a trailing newline). There is no include mechanism between them; any change must be applied to **both files** to keep them in sync.

## Development workflow

- Edit the HTML/CSS/JS directly in `Index.html` and `GlobalNYE.html`, applying the same change to both files.
- To preview, simply open the file in a browser (e.g. `open Index.html` or a "Live Server"-style extension) — no install or build step is required.
- There is no linter, formatter, or test command configured in this repo.

## Architecture

Everything lives in one script block per file:

- `timeZones`: a static array of 38 entries covering every UTC offset from +14 to -12 (including half-hour and 45-minute offsets like Nepal's +5:45), each with `name`, `offset`, and `label`.
- `activeTimers`: the in-memory list (max 5) of currently displayed timezones; seeded by default with the first (`Kiribati`) and last (`Baker Island`) entries in `timeZones`.
- `addTimer()` / `removeTimer()`: mutate `activeTimers` from the dropdown `<select>` + ADD button and per-card ✕ button, then call `renderTimers()`. Duplicates are prevented by matching on `name`; adding beyond 5 shows an `alert()`.
- `renderTimers()`: rebuilds the card DOM in `#timerContainer` from scratch on every add/remove (full re-render, not diffed).
- `updateCountdowns()`: runs every second via `setInterval`, computing each timezone's local-midnight instant as `Date.UTC(2026, 0, 1, 0, 0, 0) - (offset * 3600000)` and rendering the H/M/S remaining, or a flashing "HAPPY NEW YEAR" message once the diff hits zero.

The target year (`2026`) and the "Current Date" text in the footer are hardcoded and would need manual updates for future years.

DOM elements are keyed by sanitized timezone name (spaces stripped) for both the card `id` and the countdown display `id`, e.g. `timer-Tokyo,Seoul,Pyongyang` / `display-Tokyo,Seoul,Pyongyang`.
