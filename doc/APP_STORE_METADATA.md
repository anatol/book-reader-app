# App Store Metadata: Booklight

Use this file when creating the app record in App Store Connect.

## 1) Core Listing Text

- **App Name**: `Booklight`
- **Subtitle** (max 30 chars): `Private PDF & EPUB Reader`
- **Promotional Text** (max 170 chars):
  `A simpler, more stable reading experience than the official app. Continue on MacBook at work and iPad in the park with synced position and no iCloud space.`
- **Description**:

  ```text
  Booklight is a fast, private reader for your own PDF and EPUB library, built for a simpler and more stable reading experience than the official app.

  Bring your own folders, open books instantly, and keep reading with a clean interface built for long sessions.

  Features:
  - PDF and EPUB support
  - Automatic reading-position tracking
  - Active Books list sorted by recency
  - Fast local library scanning and caching
  - Fuzzy title search
  - Works fully offline

  Booklight does not require an account and does not send analytics or telemetry.

  Your files remain in your storage locations. Booklight reads from selected folders and stores reading progress locally.

  Read the same book across devices: start on a MacBook at the office and continue on an iPad in the park at the exact saved position.

  Booklight does not require iCloud and does not consume iCloud storage. Use any sharing mechanism you prefer, including Syncthing, to synchronize books and reading position across devices.
  ```

## 2) Discoverability

- **Keywords** (max 100 chars total):
  `ebook,epub,pdf,reader,books,library,offline,privacy,study,document`
- **Primary Category**: `Books`
- **Secondary Category**: `Productivity` (optional)

## 3) URLs Required by App Store Connect

Set these to your real hosted pages before submission:

- **Support URL**: `https://YOUR-DOMAIN/booklight/support`
- **Marketing URL** (optional): `https://YOUR-DOMAIN/booklight`
- **Privacy Policy URL**: `https://YOUR-DOMAIN/booklight/privacy`

Template content for privacy policy is in `doc/PRIVACY_POLICY.md`.

## 4) App Privacy (Nutrition Label)

Suggested answers based on current codebase behavior:

- **Data Used to Track You**: `No`
- **Data Linked to You**: `No`
- **Data Not Linked to You**: `No`
- **Third-Party Advertising**: `No`
- **Analytics/Telemetry Collection**: `No`

Re-validate these answers if you add network calls, crash reporting, analytics SDKs, or ads.

## 5) Age Rating

Suggested rating: **4+**

- No user-generated content feed
- No gambling
- No mature themes
- No unrestricted web access

## 6) Review Information

- **Demo Account Required**: `No`
- **Sign-in Required**: `No`
- **Review Notes**:

  ```text
  The app works fully offline and does not require account creation.
  To test reading, import any PDF or EPUB from Files and open it from the library view.
  ```

## 7) Pricing and Availability

- **Price**: set in App Store Connect (Free or Paid)
- **In-App Purchases**: `None` (current project)
- **Territories**: choose as desired

## 8) Versioning Checklist

Current project values (already configured):

- `MARKETING_VERSION = 1.0`
- `CURRENT_PROJECT_VERSION = 1`
- `PRODUCT_BUNDLE_IDENTIFIER = com.anatol.Booklight`
- `ITSAppUsesNonExemptEncryption = NO`
- `LSApplicationCategoryType = public.app-category.books`

Before each release:

1. Increment `MARKETING_VERSION` for user-visible release version.
2. Increment `CURRENT_PROJECT_VERSION` for build number.
3. Add release notes in App Store Connect “What’s New”.

## 9) Assets Checklist

- App Icon 1024x1024: present in `AppIcon.appiconset/icon-1024.png`
- Screenshots for required device classes:
  - iPad (required if shipping iPad app)
  - Mac (for Mac Catalyst listing)

Candidate screenshots exist in `doc/screenshots/`, but App Store Connect needs correctly sized exports.
