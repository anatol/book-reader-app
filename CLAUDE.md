# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Style

Add comments to the code that is not clear from the context.

## Build Commands

**iOS (compile-only verification):**
```bash
xcodebuild -project Booklight.xcodeproj -scheme Booklight -destination 'generic/platform=iOS' -derivedDataPath /tmp/BooklightDerived CODE_SIGNING_ALLOWED=NO build
```

**Mac Catalyst:**
```bash
xcodebuild -project Booklight.xcodeproj -scheme Booklight -destination 'generic/platform=macOS,variant=Mac Catalyst' -derivedDataPath /tmp/BooklightDerived-catalyst CODE_SIGNING_ALLOWED=NO build
```

Use `-derivedDataPath /tmp/...` to avoid permissions issues. `CODE_SIGNING_ALLOWED=NO` for CI-like builds.

## Tests

No automated tests exist yet. Validation is compile-time only via `xcodebuild`.

## Full Developer Documentation

See [doc/DEVELOPMENT.md](doc/DEVELOPMENT.md) for architecture details, manual test checklist, troubleshooting, and Homebrew release instructions.
