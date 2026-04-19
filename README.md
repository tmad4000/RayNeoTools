# RayNeo Tools

A CLI-controllable Android browser for the RayNeo X3 Pro AR glasses, plus a host-side CLI (`rn`) that drives it over ADB.

## Why

TapLink (the default browser on RayNeo X3) has a custom on-screen keyboard that ignores `adb shell input text`, and it doesn't register an `android.intent.action.VIEW` handler, so URLs can't be opened programmatically. This repo ships:

- **`browser/`** — `com.jacobcole.rayneobrowser`: a minimal Android WebView browser using standard widgets (works with `adb input text`, tab focus nav, and `VIEW` intents) plus a localhost HTTP control server on port `7317`.
- **`cli/rn`** — Mac-side CLI wrapping `adb` + the browser's HTTP API.

## Quick start

```bash
# 1. Build & install the APK (requires JDK 17, Android SDK at /opt/homebrew/share/android-commandlinetools)
JAVA_HOME=/opt/homebrew/opt/openjdk@17 ./gradlew :app:assembleDebug
./cli/rn install

# 2. Use it
./cli/rn open https://google.com
./cli/rn type "core balance training"
./cli/rn key ENTER
./cli/rn shot          # /tmp/rayneo.png
./cli/rn title
./cli/rn back
./cli/rn js 'document.querySelectorAll("a").length'
```

Symlink for global use:
```bash
ln -sf "$(pwd)/cli/rn" /usr/local/bin/rn
```

## CLI commands

| Cmd | Purpose |
|-----|---------|
| `rn open URL` | Launch browser with `VIEW` intent |
| `rn type TEXT` | `adb shell input text` with shell-metachar escaping |
| `rn key KEY` | Keyevent (`TAB`, `ENTER`, `BACK`, `DPAD_DOWN`, …) |
| `rn tap X Y` | Tap screen coordinate |
| `rn swipe X1 Y1 X2 Y2 [MS]` | Swipe gesture |
| `rn shot [OUT]` | Screenshot PNG (default `/tmp/rayneo.png`) |
| `rn js 'CODE'` | Eval JS in WebView (returns result) |
| `rn dom` | Dump rendered DOM |
| `rn title` | Current page title + URL (JSON) |
| `rn back` / `rn fwd` | Browser back / forward |
| `rn ping` | Health check the control server |
| `rn install` | Install/reinstall APK |
| `rn forward` | Set up `adb forward tcp:7317` |

## HTTP Control API (on glasses)

Runs on `127.0.0.1:7317` inside the browser app. Accessible from Mac after `adb forward tcp:7317 tcp:7317`.

- `POST /url` — JSON `{url}` → navigate
- `POST /js` — raw JS body → eval, return JSON result
- `GET /dom` — rendered HTML
- `GET /screenshot` — PNG of decor view
- `GET /back` / `GET /forward` — nav
- `GET /title` — `{title, url}`
- `GET /ping` — health

## Tested workflows

- Google search
- YouTube (loads video pages)
- `corebalancetraining.com` login + post pages

## Known limitations

- The RayNeo system sometimes overlays a settings dock when switching apps. Launching the browser again via `rn open` generally brings it to front.
- Text input uses `adb shell input text`; special chars beyond the standard escape set may still need manual handling.
- Screenshot captures the browser's own `decorView`, not the entire display (so system overlays aren't captured). For full-display capture use `adb exec-out screencap -p`.

## Project layout

```
RayNeoTools/
├── browser/app/…         # Android app (Kotlin)
├── cli/rn                 # Mac CLI
├── build.gradle.kts       # root Gradle (AGP 8.7, Kotlin 2.0)
├── settings.gradle.kts
├── gradle.properties
├── gradle/wrapper/
└── .beads/                # issue tracking (bd)
```

## Beads tickets

Tracked under prefix `rn-`:
- `rn-be7` (epic) — umbrella
- `rn-db1` — scaffold browser ✓
- `rn-2c4` — HTTP control server ✓
- `rn-0ie` — `rn` CLI ✓
- `rn-881` — install + verify ✓
- `rn-cf8` — YouTube-focused viewer (follow-on)
- `rn-48z` — AccessibilityService bridge (unlocks TapLink control)
