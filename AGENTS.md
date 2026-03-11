# AGENTS.md

## Project overview

Safari Profile Router is a native macOS SwiftUI menu bar app. It registers as the default browser, intercepts URLs, matches them against user-defined glob pattern rules, and opens them in the correct Safari profile using AppleScript.

## Tech stack

- **Language:** Swift (SwiftUI + AppKit)
- **Build:** `swiftc` via `build.sh` — no Xcode project. Produces a universal binary (arm64 + x86_64).
- **Target:** macOS 14.0+ (Sonoma) — requires Safari profiles
- **CI:** GitHub Actions (`build.yml` for PRs, `release.yml` for tagged releases)
- **Data:** UserDefaults for rules, profiles, and settings
- **Logging:** `~/Library/Logs/URLRouter.log`

## Key patterns

- **URL interception:** `NSAppleEventManager` handles `kInternetEventClass`/`kAEGetURL` in `AppDelegate`
- **Pattern matching:** `NSPredicate(format: "SELF LIKE %@")` for glob matching in `Rule.matches(url:)`
- **Safari profile control:** Runs AppleScript via `osascript` (`Process`) to click Safari's File > New Window > "New {Profile} Window" menu item through System Events. Requires Accessibility permission.
- **Fallback:** If AppleScript fails, opens directly in Safari via `NSWorkspace.shared.open([url], withApplicationAt:)` to avoid an infinite loop (since URLRouter is the default browser, `NSWorkspace.shared.open(url)` would route back to itself)

## Build and run

```bash
./build.sh          # Build app
./build.sh --dmg    # Build app + create DMG
cp -r build/URLRouter.app /Applications/
```

## Known constraints

- Ad-hoc code signing means Accessibility permission must be re-granted after each rebuild/reinstall
- Safari profile discovery requires Safari to be running (it reads menu items via AppleScript)
- URLs with `/` in the pattern match against the full URL string; without `/` they match hostname only
