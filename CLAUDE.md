# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**React Wordle** — a clone of the popular Wordle word-guessing game built with React 17, TypeScript, and Tailwind CSS. It's a Create React App (CRA) project with one daily puzzle solution derived from an epoch-based index.

## Key Commands

```bash
npm run start       # Dev server (localhost:3000)
npm run build       # Production build
npm run test        # Run tests (CRA watch mode, jest)
npm run lint        # Check formatting (prettier)
npm run fix         # Auto-fix formatting (prettier --write)
```

## Architecture

### Entry Point

- `src/index.tsx` — mounts `<App>` wrapped in `<AlertProvider>` (the sole React Context)

### Main App (`src/App.tsx`)

The single-page app manages all game state at the top level:
- **Game state**: `guesses` (array of submitted words), `currentGuess`, `isGameWon`, `isGameLost`
- **Settings**: `isHardMode`, `isDarkMode`, `isHighContrastMode` (persisted to localStorage)
- **UI**: Modal open/close state, current row animation class, reveal animation lock (`isRevealing`)
- **Persistence**: Game state saved to localStorage on every guess; restored on load (only if the solution matches today's)

### Component Structure

```
src/components/
├── alerts/          — AlertContainer + Alert (toast notifications via AlertContext)
├── grid/            — Grid, Cell, CompletedRow, CurrentRow, EmptyRow (guess display)
├── keyboard/        — Keyboard + Key (virtual keyboard with color-coded status)
├── modals/          — BaseModal (Dialog from @headlessui), InfoModal, StatsModal, SettingsModal, SettingsToggle
├── navbar/          — Navbar (title + icon buttons for modals)
└── stats/           — StatBar, Histogram, Progress (statistics display)
```

### Game Logic (`src/lib/`)

| File | Purpose |
|------|---------|
| `words.ts` | Word validation, hard-mode enforcement, unicode-aware splitting (grapheme-splitter), daily solution via `getWordOfDay()` |
| `statuses.ts` | Computes letter statuses (correct/present/absent) for a guess |
| `stats.ts` | Load/save game statistics (wins, losses, streaks, distribution) |
| `share.ts` | Generate emoji grid share text |
| `localStorage.ts` | Load/save game state and settings to localStorage |

### Constants (`src/constants/`)

| File | Purpose |
|------|---------|
| `settings.ts` | `MAX_WORD_LENGTH` (5), `MAX_CHALLENGES` (6), timing constants |
| `strings.ts` | All user-facing strings (for i18n) |
| `wordlist.ts` | `WORDS` — the list of valid solutions |
| `validGuesses.ts` | `VALID_GUESSES` — broader list of acceptable guesses |

### Daily Solution Logic

The solution is deterministically derived from the number of days since January 1, 2022, modulo the WORDS array length. This means the same wordlist always yields the same daily word for all players.

### Environment Variables (`.env`)

- `REACT_APP_GAME_NAME` — title displayed in the navbar
- `REACT_APP_GAME_DESCRIPTION` — meta description
- `REACT_APP_LOCALE_STRING` — locale for language-aware case conversion (enables i18n support)
- `REACT_APP_GOOGLE_MEASUREMENT_ID` — Google Analytics (optional)

### Styling

- **Tailwind CSS** for utility classes
- **Dark mode** via `dark` class on `<html>` element
- **High contrast mode** via `high-contrast` class on `<html>` element
- Custom CSS in `src/App.css` and `src/index.css` for animations (jiggle, bounce)

### Dependencies

- `@headlessui/react` — modal/dialog components
- `@heroicons/react` — icon library
- `grapheme-splitter` — proper unicode grapheme handling for non-ASCII characters
- `react-countdown` — countdown timer (time until next puzzle)
- `classnames` — conditional CSS class composition

### Testing

Uses CRA's built-in Jest + React Testing Library setup. The test file is `src/App.test.tsx`. Run with `npm run test` (watch mode) or `CI=true npm run test` (single run).
