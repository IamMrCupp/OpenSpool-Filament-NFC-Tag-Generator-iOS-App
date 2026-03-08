# NFC Tag Writer/Reader — iOS App Technical Plan

## Project Overview

An iOS application for reading and writing custom NFC tags, built with SwiftUI targeting iOS 14+. The app will support custom/app-specific NDEF payloads with both read and write capabilities.

---

## My Recommendations

### SwiftUI (chosen)
Since you're not locked to UIKit, SwiftUI is the right call. It's Apple's current direction, results in less boilerplate, and pairs well with modern Swift concurrency (`async/await`). NFC sessions use delegate callbacks which we'll wrap cleanly into an `ObservableObject`.

### iOS 14 Minimum (chosen)
Good choice. This gives us background tag reading on capable devices and a large user base.

---

## 1. Developer Account & Entitlements

Before a single line of code works, you'll need:

- **Paid Apple Developer account** ($99/yr) — NFC writing is not available on free accounts
- **App ID** with `Near Field Communication Tag Reading` entitlement enabled in the Developer Portal
- **Provisioning profile** that includes the NFC entitlement
- In Xcode: `Signing & Capabilities` → add `Near Field Communication Tag Reading`
- In `Info.plist`: add `NFCReaderUsageDescription` with a human-readable string explaining NFC use

---

## 2. NFC Capabilities & Tag Compatibility

For **custom/app-specific data**, you'll be working with:

- **NDEF format** with record type `NFCNDEFPayload` using type name format `NFCTypeNameFormat.nfcExternal` — this is Apple's mechanism for custom payloads
- The external type string follows the format: `your.domain:typename` (e.g., `com.yourapp:profile`)
- **Recommended physical tags**: NTAG213 (144 bytes), NTAG215 (504 bytes), NTAG216 (888 bytes) — widely available, fully NDEF-compatible, rewritable

---

## 3. Project Structure

```
NFCApp/
├── NFCAppApp.swift               # App entry point
├── ContentView.swift             # Root navigation
│
├── NFC/
│   ├── NFCManager.swift          # Core ObservableObject — owns all NFC sessions
│   ├── NFCReader.swift           # NFCNDEFReaderSession delegate logic (read)
│   ├── NFCWriter.swift           # NFCNDEFReaderSession delegate logic (write)
│   └── NDEFPayloadBuilder.swift  # Constructs custom NDEF records from your data model
│
├── Models/
│   └── AppTagPayload.swift       # Your custom data model (what gets written/read)
│
├── Views/
│   ├── HomeView.swift
│   ├── WriteTagView.swift        # UI for composing and writing a tag
│   ├── ReadTagView.swift         # UI for displaying scanned tag content
│   └── ScanAnimationView.swift   # Visual feedback during scan
│
└── Utilities/
    └── NFCError.swift            # Unified error enum for NFC failures
```

---

## 4. Core Architecture

**`NFCManager`** is the heart of the app — a singleton `ObservableObject` that:
- Vends `@Published` state (`isScanning`, `lastReadPayload`, `errorMessage`)
- Creates and invalidates `NFCNDEFReaderSession` instances
- Routes delegate callbacks to either read or write handling logic
- Serializes/deserializes your `AppTagPayload` model to/from raw bytes

### Session Flow

```
User taps "Write" →
  NFCManager.startWriteSession() →
    NFCNDEFReaderSession(delegate: NFCWriter, ...) →
      System NFC sheet appears →
        User taps tag →
          readerSession(_:didDetect:) fires →
            Write NDEF message →
              Invalidate session with success message
```

The same flow applies for reading, ending in parsing bytes back into `AppTagPayload`.

---

## 5. Custom Payload Design

Your `AppTagPayload` needs to serialize to/from `Data`. Plan to define:

```swift
struct AppTagPayload: Codable {
    let version: Int               // for forward compatibility
    let type: String               // what kind of record this is
    let payload: [String: String]  // flexible key-value data
}
```

**Serialization strategy**: `JSONEncoder` → `Data` → written as the NDEF payload body. Clean, versioned, and extensible.

The NDEF record will use:
- **Type Name Format**: `.nfcExternal`
- **Type**: `com.yourapp:appdata` (you define this string — replace with your actual bundle ID domain)
- **Payload**: raw JSON bytes

---

## 6. Error Handling Strategy

NFC has many failure modes. Define a unified error enum covering:

| Error Case | Description |
|---|---|
| `tagNotCompatible` | Tag doesn't support NDEF |
| `tagTooSmall(required: Int, available: Int)` | Payload exceeds tag capacity |
| `sessionTimeout` | User didn't scan in time |
| `writeProtected` | Tag is locked |
| `invalidPayload` | Deserialization failed on read |
| `sessionInvalidated(reason: String)` | Catch-all for CoreNFC errors |

All errors surface through `NFCManager`'s `@Published var errorMessage: String?` so the UI reacts automatically.

---

## 7. Key Constraints & Gotchas

| Constraint | Detail |
|---|---|
| **No simulator support** | Physical iPhone required for all NFC testing |
| **One session at a time** | You cannot have concurrent read+write sessions |
| **Session timeout** | ~60 seconds; user must tap tag within this window |
| **NDEF only for custom data** | Raw binary tag access not available for most tag types |
| **Tag locking** | Once locked, a tag cannot be rewritten — expose this carefully in UI |
| **iPhone 7+** | NFC hardware minimum; background reading needs iPhone XS+ / iOS 14 |
| **Alert message required** | `alertMessage` on the session is shown to the user by iOS — make it helpful |

---

## 8. Testing Plan

Since there's no simulator support:

1. **Unit test** `NDEFPayloadBuilder` and `AppTagPayload` serialization — no hardware needed
2. **Unit test** error mapping logic
3. **Device test** with physical NTAG2xx tags (cheap, ~$0.50–$1 each on Amazon)
4. **Edge case testing**: tag too small, locked tag, session timeout, malformed existing data on tag

---

## 9. Recommended Implementation Order

1. Project setup, entitlements, and provisioning
2. `AppTagPayload` model + serialization (fully testable without hardware)
3. `NDEFPayloadBuilder` — encode/decode logic
4. `NFCManager` skeleton with session lifecycle
5. `NFCWriter` delegate — write flow end-to-end
6. `NFCReader` delegate — read flow end-to-end
7. SwiftUI views wired to `NFCManager`
8. Error handling polish
9. Edge case testing on hardware

---

## Open Questions / Annotation Space

> Use this section to capture decisions, questions, and clarifications during review.

- [ ] What will the `com.yourapp` domain be? (needed for NDEF external type string)
- [ ] What specific key-value fields will `AppTagPayload.payload` carry?
- [ ] Do we need to support reading tags written by other apps / non-NDEF tags?
- [ ] Should tags be writable only once (locked after first write), or always rewritable?
- [ ] Do we need any cloud sync or persistence of read tag history?
- [ ] What is the target App Store distribution, or is this internal/enterprise?
