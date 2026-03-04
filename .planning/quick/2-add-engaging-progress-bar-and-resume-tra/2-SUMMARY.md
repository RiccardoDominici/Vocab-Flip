---
phase: quick-2
plan: 01
subsystem: ui/progress
tags: [flutter, progress, animation, home-screen, spaced-repetition]
dependency_graph:
  requires: []
  provides: [progress-hero, resume-tracking, overall-stats]
  affects: [lib/screens/home_screen.dart, lib/services/progress_service.dart]
tech_stack:
  added: []
  patterns: [CustomPainter, TweenAnimationBuilder, SweepGradient, SharedPreferences]
key_files:
  created: []
  modified:
    - lib/services/progress_service.dart
    - lib/screens/home_screen.dart
decisions:
  - "Used TweenAnimationBuilder instead of AnimationController for simpler animated ring with no dispose overhead"
  - "Deferred the widget test failure as pre-existing (pending timer issue with flutter_animate already present before this task)"
metrics:
  duration: "2m 35s"
  completed: "2026-03-04"
  tasks_completed: 2
  files_modified: 2
---

# Phase quick-2 Plan 01: Progress Bar and Resume Training Summary

Animated circular progress ring and resume button added to the home screen, backed by last-played theme tracking and CEFR-level overall stats in ProgressService.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Add last-played tracking and overall stats to ProgressService | 81969e0 | lib/services/progress_service.dart |
| 2 | Build progress hero section with animated ring and resume button | 3f8b71e | lib/screens/home_screen.dart |

## What Was Built

### Task 1 — ProgressService enhancements

- `String? _lastPlayedTheme` field loaded from SharedPreferences key `last_played_theme` during `_loadAll()`
- `getLastPlayedTheme()` — sync getter for the stored theme name
- `saveLastPlayedTheme(String themeName)` — persists to SharedPreferences; auto-called from `completeRound()`
- `resetAllProgress()` — now also clears `last_played_theme` key and resets `_lastPlayedTheme = null`
- `getOverallStats(String cefrLevel)` — computes `totalWords`, `masteredWords`, `knownWords`, `seenWords`, `overallCompletion` across all themes matching the given CEFR level
- `getResumeTheme(String cefrLevel)` — returns the best theme to resume: last-played if in-progress, else lowest-completion in-progress theme, else first 0% theme, else first theme
- `OverallStats` data class added at end of file

### Task 2 — Home screen progress hero

- Replaced "X categorie - Y parole" text with `_ProgressHero` widget
- `_ProgressHero` — glass card with animated circular ring (left) + stats/button column (right)
- `_ProgressRingPainter` — `CustomPainter` drawing background track + foreground SweepGradient arc with rounded caps, starting at 12 o'clock (-pi/2)
- `TweenAnimationBuilder<double>` animates ring from 0 to `overallCompletion` over 800ms with `Curves.easeOutCubic`
- Center of ring shows percentage number (22sp bold) + "%" label (12sp)
- Mini stats row: green dot mastered / blue dot known / gray dot remaining using `_StatDot` helper
- Gradient resume button: "Riprendi" + theme name when last-played exists; "Inizia" + theme name otherwise
- Navigates to `GameScreen` with `FadeTransition`, calls `onReturn()` after pop
- Entire hero wrapped with `.animate().fadeIn(delay: 350.ms).slideY(begin: 0.1)`

## Deviations from Plan

### Pre-existing issue (out of scope)

**Widget test failure** — `flutter test` was already failing before this task with a pending timer assertion caused by `flutter_animate` in the existing smoke test. Confirmed by stashing our changes and running the test on the base commit. Logged here for visibility but not fixed (out of scope per deviation rules).

**Deferred to `deferred-items.md`:** Fix widget test by pumping animation frames (`await tester.pumpAndSettle()`) or using `flutter_animate`'s test utilities.

## Self-Check

- [x] `lib/services/progress_service.dart` — exists and modified
- [x] `lib/screens/home_screen.dart` — exists and modified
- [x] Commit 81969e0 — exists (ProgressService)
- [x] Commit 3f8b71e — exists (HomeScreen)
- [x] `flutter analyze` — No issues found
