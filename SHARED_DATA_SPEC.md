# Cross-Platform Shared Data Specification

## 1. Overview

**Google Meet Desktop & Mobile Manager** uses a standardized, vendor-neutral JSON format (Schema Version 2) to export and restore permanent meeting buttons between:
- **Windows Desktop** (Java 17 Swing + SQLite)
- **Android Mobile** (Kotlin + Jetpack Compose + Room)
- **Web Companion & Simulator** (React + LocalStorage)

> **Important Operational Note:** This format is designed for offline, manual backup, migration, and restoration. It is **not** an automated real-time cloud synchronization service. Meeting records remain strictly private and local to each device unless exported by the user.

---

## 2. Schema Definition (Version 2)

```json
{
  "schemaVersion": 2,
  "exportTimestamp": 1729432000000,
  "appName": "Google Meet Desktop & Mobile Manager",
  "records": [
    {
      "id": "e3b0c442-98fc-1c14-9afb-4c8996fb9242",
      "name": "Daily Engineering Standup",
      "url": "https://meet.google.com/abc-defg-hij",
      "category": "Daily Standup",
      "description": "Daily team sync every weekday at 09:30 AM",
      "notes": "Bring blockers and PR updates",
      "scheduledTime": "Mon-Fri 09:30 AM",
      "favorite": true,
      "color": "Google Blue",
      "joinCount": 12,
      "lastJoinedAt": 1729430000000,
      "createdAt": 1729400000000,
      "updatedAt": 1729430000000
    }
  ]
}
```

### Field Definitions:

| Field | Type | Required | Description |
|---|---|---|---|
| `schemaVersion` | Integer | Yes | Format version number (`2`). |
| `exportTimestamp` | Long (Epoch ms) | Yes | Timestamp of when the backup file was generated. |
| `appName` | String | Yes | Name of exporting application. |
| `records[].id` | String (UUID) | Yes | Unique identifier for meeting button. |
| `records[].name` | String | Yes | Display name on button (non-empty). |
| `records[].url` | String | Yes | Normalized HTTPS Google Meet URL (`https://meet.google.com/xxx-yyyy-zzz`). |
| `records[].category` | String | No | Category grouping tag (defaults to `"General"` if blank). |
| `records[].description`| String | No | Optional human-readable description or purpose. |
| `records[].notes` | String | No | Optional agenda notes or reminders. |
| `records[].scheduledTime` | String | No | Optional scheduled date/time string. |
| `records[].favorite` | Boolean | No | Whether the button is pinned to the top as a favorite. |
| `records[].color` | String | No | Visual theme color (`"Google Blue"`, `"Emerald Green"`, etc.). |
| `records[].joinCount` | Integer | No | Total times this button has launched the meeting. |
| `records[].lastJoinedAt` | Long | No | Timestamp of the last time this meeting was launched. |
| `records[].createdAt` | Long | Yes | Epoch timestamp when created. |
| `records[].updatedAt` | Long | Yes | Epoch timestamp when last edited. |

---

## 3. Backward Compatibility with Schema v1

When importing, the parser automatically checks if the root element is:
1. **A JSON Object with `records`**: Parses the v2 envelope.
2. **A raw JSON Array of meetings**: Parses legacy v1 records and auto-upgrades them with default v2 attributes (`favorite: false`, `color: "Google Blue"`, `scheduledTime: ""`).

---

## 4. Import & Restore Modes

Both Windows and Android prompt the user before restoring data:
- **Merge Mode (Default & Safe)**: Checks both `id` and `url`. If a meeting with the same ID or same URL exists, the existing record is preserved and only new items are appended.
- **Replace All Mode**: Clears current local records and restores the exact contents of the backup. Requires explicit user confirmation.

If file validation fails or the JSON is malformed, **the import is aborted immediately and existing local data is never erased**.

---

## 5. One-Click Meeting Join & Local Settings Persistence

Both Windows Desktop and Android Mobile provide a dedicated, prominent **"JOIN MEETING NOW"** 1-click launcher on the home screen/dashboard.

### Storage Architecture:
Per security and privacy principles, default meeting settings are retained strictly **offline and local to each device** (no automated cloud sync):
- **Windows Desktop**: Stored in the SQLite `app_settings` table (`key TEXT PRIMARY KEY, value TEXT NOT NULL`) within `%APPDATA%\GoogleMeetDesktopManager\meetings.db`.
- **Android Mobile**: Stored in the Room SQLite `app_settings` entity table within the app's private sandbox database (`google_meet_manager.db`).
- **Web Simulator**: Stored in browser `localStorage` (`gmeet_default_meeting_v1`).

### Stored Keys:
- `default_meeting_title`: Human-readable label for the primary room (defaults to `"Primary Google Meet"`).
- `default_meeting_url`: Fully qualified HTTPS Google Meet URL (e.g., `https://meet.google.com/abc-defg-hij`).
- `default_meeting_last_joined`: Epoch millisecond timestamp of last one-click launch.
- `default_meeting_join_count`: Integer counter tracking cumulative 1-click launches.

### URL Validation Rule:
Only legitimate HTTPS Google Meet URLs (`https://meet.google.com/[a-z]{3}-[a-z]{4}-[a-z]{3}`) are permitted. Insecure HTTP or non-Google domains are rejected with explicit feedback.

