# Safari Profile Router

A native macOS menu bar app that routes URLs to the right Safari profile. Register it as your default browser, define pattern-matching rules, and links from Slack, Teams, email etc. automatically open in the correct profile.

## How it works

1. The app registers as the default browser on macOS (handles `http`/`https` URLs)
2. When any app opens a URL, macOS hands it to Safari Profile Router
3. The URL is matched against your rules (glob patterns, checked top to bottom)
4. The app uses AppleScript to open Safari in the matched profile — either as a new tab in an existing profile window, or by creating a new profile window via the File > New Window menu
5. If no rule matches, the URL opens in your configured default profile
6. If AppleScript fails (e.g. missing Accessibility permission), the URL falls back to opening directly in Safari's default profile

## Requirements

- macOS Sonoma (14.0+) — Safari profiles were introduced in this version
- Accessibility permission for the app (System Settings > Privacy & Security > Accessibility)

## Install

### From release

Download the DMG from the [latest release](https://github.com/bradystroud/safari-profile-router/releases/latest), open it, and drag URLRouter to Applications.

### From source

```bash
./build.sh
cp -r build/URLRouter.app /Applications/
```

Then launch URLRouter and click **Set as Default Browser**.

> **Note:** After each rebuild/reinstall, you'll need to re-add URLRouter.app to the Accessibility permission list in System Settings. The ad-hoc code signature changes on each build, which invalidates the previous permission grant.

## Usage

### Rules

Rules match URLs using glob patterns (`*` matches any characters, `?` matches a single character):

| Pattern | Matches |
|---|---|
| `*.github.com` | Any github.com subdomain |
| `*jira*` | Any URL with "jira" in the hostname |
| `*/*asfaudits*` | Any URL with "asfaudits" anywhere (full URL match) |
| `mail.google.com` | Exact hostname match |

- Patterns **without** `/` match against the **hostname** only
- Patterns **with** `/` match against the **full URL** (use this for path-based or query string matching)
- Matching is case-insensitive
- Rules are evaluated top to bottom — first match wins
- Rules can be reordered, toggled on/off, and edited in the settings UI

### Default profile

URLs that don't match any rule open in the default profile (configurable in settings).

### Profiles

Click **Refresh Profiles** in settings to discover your Safari profiles. These are read from Safari's File > New Window submenu via AppleScript. Discovered profiles are cached so the dropdown is available immediately on subsequent launches.

### Logs

The **Logs** tab in settings shows real-time routing decisions with color-coded entries:
- **MATCH** — a rule matched the URL
- **NO_MATCH** — a rule was tested but didn't match
- **ROUTE** — the final routing decision
- **ERROR** — something went wrong (e.g. AppleScript failure)

Logs are also written to `~/Library/Logs/URLRouter.log`.

## Architecture

```
URLRouter/
├── URLRouterApp.swift          # App entry, menu bar icon, URL event handler
├── Info.plist                  # URL scheme registration (http/https)
├── URLRouter.entitlements      # Apple Events automation entitlement
├── Models/
│   └── Rule.swift              # Rule model with glob pattern matching
├── Services/
│   ├── RuleEngine.swift        # URL-to-profile matching, rule persistence
│   ├── SafariLauncher.swift    # Opens URLs in Safari profiles via osascript
│   ├── ProfileDiscovery.swift  # Discovers Safari profiles from menus
│   └── Logger.swift            # File + in-memory logging
└── Views/
    ├── SettingsView.swift      # Main settings window with rules + logs tabs
    ├── RuleEditorView.swift    # Add/edit rule dialog
    └── LogView.swift           # Log viewer
```

The app is compiled with `swiftc` directly (no Xcode project needed) and produces a universal binary (arm64 + x86_64). See `build.sh`.

## Releasing

Push a version tag to create a GitHub release with a DMG attached:

```bash
git tag v1.x.x
git push --tags
```
