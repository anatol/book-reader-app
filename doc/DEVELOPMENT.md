# Development Guide

## Prerequisites

- Xcode 26 or newer
- Apple platform SDKs (installed with Xcode)

Optional:

- A physical iPhone or iPad for testing folder access and reading
- A [Syncthing](https://syncthing.net/)-shared folder for multi-device sync verification

## Getting the Source

```bash
git clone https://github.com/anatol/booklight.git
cd booklight
open Booklight.xcodeproj
```

## Building

### From Xcode

1. Open `Booklight.xcodeproj`
2. Select the **Booklight** scheme
3. Choose a destination:
   - iPhone or iPad simulator
   - Physical iPhone/iPad
   - **My Mac (Mac Catalyst)** for the macOS build
4. **Product > Build** (or **Product > Run** to build and launch)

### Command Line — iOS

```bash
SWIFT_MODULECACHE_PATH=/tmp/swift-module-cache \
CLANG_MODULE_CACHE_PATH=/tmp/clang-module-cache \
xcodebuild \
  -project Booklight.xcodeproj \
  -scheme Booklight \
  -destination 'generic/platform=iOS' \
  -derivedDataPath /tmp/BooklightDerived \
  CODE_SIGNING_ALLOWED=NO \
  build
```

### Command Line — Mac Catalyst

```bash
SWIFT_MODULECACHE_PATH=/tmp/swift-module-cache \
CLANG_MODULE_CACHE_PATH=/tmp/clang-module-cache \
xcodebuild \
  -project Booklight.xcodeproj \
  -scheme Booklight \
  -destination 'generic/platform=macOS,variant=Mac Catalyst' \
  -derivedDataPath /tmp/BooklightDerived-catalyst \
  CODE_SIGNING_ALLOWED=NO \
  build
```

Notes:

- `-derivedDataPath /tmp/...` avoids permissions issues in restricted environments
- `CODE_SIGNING_ALLOWED=NO` skips signing for compile-only verification
- If Xcode rejects the destination string, select the Catalyst destination in Xcode once and copy the resolved string from the build logs

## Architecture

Booklight is a single-target SwiftUI app (~2,200 lines of Swift) that runs on iPadOS and macOS via Mac Catalyst.

### Key Components

| File | Role |
|------|------|
| `LibraryController.swift` | Core library management: folder scanning, book discovery, progress tracking, state persistence |
| `Models.swift` | Data types: `Book`, `BookProgressState`, `BookFormat`, hash-cache records, PDF reading position |
| `ContentView.swift` | Main library UI with search and Active Books / All Books sections |
| `PDFReaderView.swift` | PDF reading via PDFKit |
| `EPUBReaderView.swift` | EPUB reading via WKWebView |
| `EPUBSupport.swift` | EPUB ZIP extraction and package parsing |
| `BookArtwork.swift` | Book cover/artwork extraction and caching |
| `ReaderContainerView.swift` | Container view wrapping the format-specific readers |
| `ReaderSearch.swift` | Shared search bar UI used by both PDF and EPUB readers |
| `BooklightApp.swift` | App entry point |

### Data Persistence

The tracking directory stores active book copies separately from per-book progress JSON files. This design enables cross-device sync via file-sync tools without requiring the full local library to be shared.

```text
<Tracking Directory>/
  books/
    Book 1.pdf
    Book 2.epub
  progress/
    <book-id>.json        # Per-book reading state
```

### Progress Merge Policy

When two devices have conflicting progress for the same book, the state with the newest `updatedAt` timestamp wins. This preserves intentional backward scrolling or re-reading on another device. The `lastOpenedAt` timestamp is merged independently as the latest value.

### Project Layout

```text
Booklight.xcodeproj
Booklight/
  BooklightApp.swift
  ContentView.swift
  LibraryController.swift
  Models.swift
  ReaderContainerView.swift
  PDFReaderView.swift
  EPUBReaderView.swift
  EPUBSupport.swift
  BookArtwork.swift
  ReaderSearch.swift
  Assets.xcassets/
doc/
  DEVELOPMENT.md
```

## Testing

### Current Status

There are focused unit tests for progress normalization, merge policy, canonical progress filenames, path containment, and library sorting. Broader reader and UI workflows are still validated manually.

Suggested areas for future automated tests:

- Unit tests for EPUB package parsing in `EPUBSupport`
- UI smoke tests for library selection, search, and opening a book

### Manual Test Setup

Prepare a test folder:

```text
TrackingDirectory/
TestLibrary/
  Sample PDF.pdf
  Sample EPUB.epub
```

Select a separate tracking directory during first launch. When a book is opened, the app creates `books/` and `progress/` inside that tracking directory.

### Manual Test Checklist

#### 1. First Launch

1. Launch the app.
2. Confirm the empty state asks for a tracking directory.
3. Choose `TrackingDirectory/` as the tracking directory.
4. Open Settings and add `TestLibrary/` as a local library.
5. Confirm the app shows discovered books.

Expected: the selected tracking directory is active, and books from the local library appear in the library view.

#### 2. Library Scan

1. Add a new PDF or EPUB to the library folder.
2. Wait a few seconds or tap Refresh.
3. Remove a file from the folder.
4. Wait or tap Refresh again.

Expected: new books appear and deleted books disappear after the next scan.

#### 3. PDF Reading Progress

1. Open a PDF and move to a later page.
2. Return to the library.
3. Reopen the same PDF.

Expected: book appears in Active Books with progress shown, reopening restores the saved page.

#### 4. EPUB Reading Progress

1. Open an EPUB and scroll within a chapter.
2. Navigate to a later chapter.
3. Return to the library.
4. Reopen the EPUB.

Expected: the app restores the saved chapter and approximate scroll position.

#### 5. Active Books Ordering

1. Open several books and advance each slightly.
2. Return to the home screen after each.

Expected: active books are ordered by most recently opened. Unread or finished books stay in the lower section.

#### 6. Fuzzy Search

1. Type a partial title in the search field (e.g. `har pot` for "Harry Potter").
2. Try an inexact sequence (e.g. `hobt` for "The Hobbit").
3. Clear the search field.

Expected: matching titles remain visible, non-matching titles are filtered, clearing restores the full list.

#### 7. In-Document Search (PDF and EPUB)

1. Open a PDF or EPUB and note the current reading position.
2. Press **Cmd+F** — the search bar should appear at the top-right.
3. Type a word and press **Enter** — matches should highlight and the view should jump to the first match.
4. Press **Enter** again or click the down arrow — should navigate to the next match (wraps around).
5. Click the up arrow — should navigate to the previous match.
6. Press **Escape** or click the X button — the search bar should close and the view should return to the original reading position (from step 1).
7. Press **Cmd+F** again — the previous search term should appear pre-selected in the search bar.
8. Start typing a new word — it should replace the previous term.
9. Press left/right arrow key instead — should deselect and allow editing the previous term.

Expected: search highlights are visible (yellow for all matches, blue/orange for the current match). Match count is displayed as "X of Y". Position is restored on dismiss. Search term persists per-book within the session.

#### 8. Cross-Device Sync

1. Put the same tracking directory under Syncthing on two devices.
2. Open the same book on device A and advance further.
3. Wait for Syncthing to sync the tracking directory.
4. Open or refresh the library on device B.

Expected: synced progress appears on device B. If both devices changed progress, the newest saved state wins.

## Releasing a macOS Build via Homebrew

The macOS app is distributed through a Homebrew tap at [anatol/homebrew-tap](https://github.com/anatol/homebrew-tap).

### Step 1: Build the Release Binary

```bash
xcodebuild \
  -project Booklight.xcodeproj \
  -scheme Booklight \
  -configuration Release \
  -destination 'generic/platform=macOS,variant=Mac Catalyst' \
  -derivedDataPath /tmp/BooklightRelease \
  build
```

The `.app` bundle will be at:
```
/tmp/BooklightRelease/Build/Products/Release-maccatalyst/Booklight.app
```

### Step 2: Notarize

Create a ZIP of the app, submit for notarization, and staple the result:

```bash
# Create ZIP for notarization
cd /tmp/BooklightRelease/Build/Products/Release-maccatalyst
zip -r Booklight.zip Booklight.app

# Submit to Apple notary service
xcrun notarytool submit Booklight.zip \
  --apple-id <YOUR_APPLE_ID> \
  --team-id <YOUR_TEAM_ID> \
  --password <APP_SPECIFIC_PASSWORD> \
  --wait

# Staple the notarization ticket to the app
xcrun stapler staple Booklight.app

# Re-zip after stapling (this is the distributable archive)
zip -r Booklight.zip Booklight.app
```

If you are not using Developer ID signing, users will need to right-click the app and select "Open" to bypass Gatekeeper on first launch.

### Step 3: Create a GitHub Release

```bash
# Tag the release commit
git tag v1.0.0
git push origin v1.0.0
```

Create a release on GitHub and attach `Booklight.zip` as a release asset.

### Step 4: Update the Homebrew Formula

In the [anatol/homebrew-tap](https://github.com/anatol/homebrew-tap) repository, update the Booklight formula:

1. Set the `url` to the GitHub release asset URL
2. Update the `sha256` checksum:
   ```bash
   shasum -a 256 Booklight.zip
   ```
3. Bump the version number
4. Push to the tap repository

Verify the update:

```bash
brew update
brew install anatol/tap/booklight   # or: brew upgrade booklight
```

## Known Limitations

- No automated tests — validation is compile-time and manual only
- EPUB rendering is intentionally minimal (local ZIP extraction + WKWebView)
- In-document search finds matches within single text nodes only (cross-element matches like `<b>hel</b>lo` are not detected)
- The app assumes a flat library directory (no nested subfolders)
- Book identity is filename-based — renaming a file looks like a new book

## Troubleshooting

### Build Fails — DerivedData Permissions

Use a writable derived data path:

```bash
xcodebuild ... -derivedDataPath /tmp/BooklightDerived
```

### Build Fails — Code Signing

For compile-only verification, disable signing:

```bash
xcodebuild ... CODE_SIGNING_ALLOWED=NO build
```

### Library Does Not Update

- Wait a few seconds for the periodic rescan
- Tap the Refresh button
- Confirm new files are directly in the selected folder, not in subfolders

### EPUB Does Not Open

Possible causes: malformed EPUB archive, unsupported compression, or unusual package structure. Check the Xcode console for parser/extraction errors.
