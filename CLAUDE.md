# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fossify Gallery is a privacy-focused Android photo/video gallery app (Kotlin). It is part of the [FossifyOrg](https://github.com/FossifyOrg) ecosystem and depends heavily on the shared `org.fossify:commons` library for base classes, extensions, dialogs, and theming.

**Package:** `org.fossify.gallery` | **Min SDK:** 26 | **Target SDK:** 36

## Workflow

Always commit changes after completing work — do not ask for permission to commit.

## Build Commands

```bash
# Debug build (foss flavor)
./gradlew assembleFossDebug

# Release build
./gradlew assembleFossRelease

# Run unit tests
./gradlew testFossDebugUnitTest

# Lint check
./gradlew lintFossDebug

# Detekt static analysis
./gradlew detektMain
```

**Product flavors:** `foss` (F-Droid/open source) and `gplay` (Google Play). Always specify a flavor when building (e.g., `assembleFossDebug`, not `assembleDebug`).

**Signing:** Release builds require either a `keystore.properties` file (see `keystore.properties_sample`) or `SIGNING_*` environment variables.

## Architecture

### Core Pattern: Activities + Fragments + Room

The app uses an Activity-centric architecture (no Navigation Component). Key activity hierarchy:

- **MainActivity** → Directory browsing (implements `DirectoryOperationsListener`)
- **MediaActivity** → Media grid within a folder (implements `MediaOperationsListener`)
- **ViewPagerActivity** → Swipeable media viewer using `MyPagerAdapter` with `PhotoFragment`/`VideoFragment`
- **EditActivity** / **SetWallpaperActivity** → Extend `BaseCropActivity`
- **PhotoActivity** / **VideoActivity** → Extend `PhotoVideoActivity`
- All activities use **view binding** (not Compose, not data binding)

### Data Layer

- **Room database** (`gallery.db`, version 10) with entities: `Medium`, `Directory`, `Widget`, `DateTaken`, `Favorite`
- DAOs accessed via Context extensions: `context.mediaDB`, `context.directoryDB`, `context.favoritesDB`, `context.widgetsDB`, `context.dateTakensDB`
- **Config** (SharedPreferences wrapper) accessed via `context.config` — stores per-folder sorting, grouping, view type, and all app settings

### Media Type System

Media types use bitwise flags defined in `helpers/Constants.kt`:
`TYPE_IMAGES=1`, `TYPE_VIDEOS=2`, `TYPE_GIFS=4`, `TYPE_RAWS=8`, `TYPE_SVGS=16`, `TYPE_PORTRAITS=32`

Sorting and grouping also use bitwise flag composition (e.g., `SORT_BY_NAME or SORT_DESCENDING`).

### Image Loading

Dual image loading pipeline:
- **Glide** (primary) — custom `MyAppGlideModule` with configurable disk cache, SVG decoder support, WebP/AVIF/JXL integrations
- **Picasso** (fallback) — configured in `App.kt` with network loading disabled

### Key Extension Files

- `extensions/Context.kt` (~2000 lines) — Database access, directory sorting, media utilities
- `extensions/Activity.kt` — Activity-level helpers for media operations
- `helpers/MediaFetcher.kt` — Scans media from filesystem/MediaStore
- `helpers/Config.kt` — All SharedPreferences-backed settings with per-folder customization

## Code Style

- **Detekt** with config at `detekt.yml`, baseline at `app/detekt-baseline.xml`
- Max line length: 120 characters
- Max method length: 120 lines (Composable-annotated functions exempt)
- Max function parameters: 10 (constructors: 8)
- Max return statements: 4 (guard clauses excluded)
- Composable functions exempt from `FunctionNaming` rules
- **Lint** config at `lint.xml`, baseline at `app/lint-baseline.xml`

## Dependencies on fossify-commons

The app inherits base classes, theming, and shared UI from `org.fossify:commons`. Key inherited types:
- `FossifyApp`, `BaseSplashActivity`, `BaseConfig`, `BaseSimpleActivity`
- Shared dialogs, extensions, and RecyclerView adapter patterns

When modifying behavior that seems to come from a base class, check the commons library first.
