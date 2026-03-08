# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

iOS app for writing OpenSpool-compatible NFC tags to NTAG215/NTAG216 tags. Targets the Snapmaker U1 running the paxx12 extended firmware. The app writes an `application/json` NDEF record to the tag containing filament metadata.

## Requirements

- iOS 14.0+, iPhone 7+ (NFC hardware required)
- Xcode (latest stable recommended)
- NFC entitlement must be enabled in the app's capabilities (`com.apple.developer.nfc.readersession.formats`)

## Build & Run

Open the `.xcodeproj` or `.xcworkspace` in Xcode and run on a physical device (NFC is not available in the simulator).

```bash
# Build from command line (replace scheme/destination as appropriate)
xcodebuild -scheme OpenSpool -destination 'platform=iOS,name=My iPhone' build

# Run tests
xcodebuild -scheme OpenSpool -destination 'platform=iOS Simulator,name=iPhone 16' test

# Run a single test class
xcodebuild -scheme OpenSpool -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:OpenSpoolTests/MyTestClass test
```

## Architecture

This is a SwiftUI iOS app. Key architectural points:

- **NFC writing**: Uses `CoreNFC` (`NFCNDEFReaderSession`) to write NDEF records. The payload is `application/json` MIME type. NFC sessions must be initiated from a user action and run on a real device.
- **Tag format**: JSON payload must include required fields (`protocol`, `version`, `type`, `color_hex`) and any optional fields from the OpenSpool spec (see README for full field list).
- **Filament profiles**: Users can save/reuse profiles — persist with `UserDefaults` or a local store (implementation TBD).

## OpenSpool Tag Payload

Required fields: `protocol` (`"openspool"`), `version` (`"1.0"`), `type`, `color_hex`.

Optional fields: `brand`, `min_temp`, `max_temp`, `bed_min_temp`, `bed_max_temp`, `subtype`, `alpha`, `additional_color_hexes`, `weight`, `diameter`.

All temperature values are strings (e.g., `"230"`), not numbers.

## NFC Notes

- `NFCNDEFReaderSession` with `invalidateAfterFirstRead: false` is needed for writing.
- The app needs `NFCReaderUsageDescription` in `Info.plist`.
- NTAG215 has 504 bytes usable; NTAG216 has 888 bytes. Keep JSON payloads compact.

## Code Review Guidelines

When asked to review code or a pull request, always produce two sections in this order:

### 1. PR Summary
A concise, plain-English description of what the PR does — what problem it solves, what changed, and any notable design decisions. Intended for someone who hasn't read the diff.

### 2. Full Review
Go through the diff systematically and evaluate:

**Correctness**
- Logic errors, off-by-one issues, missing edge cases
- Proper handling of `NFCNDEFReaderSession` lifecycle (session invalidation, error callbacks, threading)
- JSON payload always includes required OpenSpool fields (`protocol`, `version`, `type`, `color_hex`)
- Temperature and numeric fields stored as strings per the spec

**Security**
- No hardcoded secrets, tokens, or PII
- User input is validated and sanitized before being written to NFC tags
- No data written to iCloud, analytics, or external services without explicit user consent
- Keychain used for any credentials; never `UserDefaults` for sensitive values
- NFC session errors surfaced to the user and not silently swallowed

**iOS-Specific**
- Entitlements and `Info.plist` keys are correct and minimal (principle of least privilege)
- UI work runs on the main thread; NFC callbacks dispatched appropriately
- No use of deprecated APIs; minimum deployment target (iOS 14.0) respected
- Memory management: no retain cycles in closures capturing `self`
- App does not request permissions it doesn't need

**Code Quality**
- Follows Swift API design guidelines (naming, types, optionals)
- No force-unwraps (`!`) unless the crash would represent an unrecoverable programmer error with a clear comment
- Errors are typed and propagated, not printed and ignored
- Tests cover new logic, especially JSON encoding/decoding and tag payload construction

Flag items as **Blocking** (must fix before merge) or **Non-blocking** (suggestion/nit).

---

## Automated PR Reviews (GitHub Actions)

The review guidelines above also run automatically on every pull request via `.github/workflows/claude-review.yml`. To activate it, add your Anthropic API key as a repository secret named `ANTHROPIC_API_KEY` in GitHub repository settings.
