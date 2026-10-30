# Consent-Based Audio Recording Specification

## 1. Overview & Policy

**Google Meet Desktop & Mobile Manager** implements a consent-verified local audio recording feature designed specifically for authorized meeting hosts, administrators, and designated note-takers.

### Core Principles:
1. **Never Record Secretly**: All recording actions require affirmative user interaction. A visible recording indicator (red pulsing dot and timer) remains prominent throughout the entire session.
2. **Explicit Consent Gate**: The user must actively check an authorization confirmation modal before recording can begin, certifying compliance with applicable state and federal wiretapping regulations (e.g. ECPA, two-party consent laws, and GDPR).
3. **Honest Capability Labeling**: The app never falsely labels a one-sided microphone capture as a complete meeting recording.

---

## 2. Preferred Recording Method: Official Google Meet Cloud Recording

The gold standard for full meeting recording is Google Meet's native cloud recording feature:

| Criteria | Google Meet Built-in Cloud Recording | Local Audio Capture Tool |
|---|---|---|
| **Audio Quality** | Full multi-track digital audio from all participants | Local microphone + system loopback |
| **Video & Screens** | Full 1080p/720p video and presentation stream | Audio-only |
| **Participant Notice** | Automated visual & audio banner to all participants | Manual host announcement required |
| **Storage** | Automatically saved to host's Google Drive | Local disk file (WAV / M4A) |
| **Prerequisites** | Google Workspace (Business Standard/Plus, Enterprise, EDU) | Any authorized host |

### How to Start Official Google Meet Recording:
1. In the active meeting window, click **Activities** (shape icon in bottom right).
2. Select **Recording**.
3. Click **Start recording** and confirm when prompted.

---

## 3. Windows Desktop Architecture (`GoogleMeetDesktopManager`)

### Audio Stream Pipeline:
- **Stream 1 (Local Microphone)**: Captures input via `TargetDataLine` using Java Sound API.
- **Stream 2 (Remote Meeting Loopback)**: Discovers system audio playback via Windows loopback lines (e.g., *Stereo Mix*, *What U Hear*, *Virtual Audio Cable*, or *WASAPI loopback*).
- **Synchronized Audio Mixer (`AudioMixer.java`)**: Combines microphone PCM and loopback PCM with dynamic headroom scaling (`(s1 * 0.9) + (s2 * 0.9)`) and soft-limiting anti-clipping protection.
- **Streaming RIFF WAV Writer (`WavFileWriter.java`)**: Streams 16-bit PCM chunks (44.1 kHz, 16-bit, mono/stereo) directly to disk, ensuring low heap memory usage during long calls.

### Storage & Safety:
- **Output Directory**: `%USERPROFILE%\Documents\GoogleMeetRecordings` (fallback: `%APPDATA%\GoogleMeetDesktopManager\recordings`).
- **Disk Space Check**: Verifies at least 100 MB free space before initializing recording.
- **Filename Pattern**: `GoogleMeet_[MeetingTitle]_[YYYY-MM-DD_HH-mm-ss].wav`.
- **Collision Protection**: Automatically increments suffix (`_1`, `_2`) if a file exists.
- **Database Catalog**: Persisted in SQLite `recordings` table.

---

## 4. Android Mobile Architecture (`GoogleMeetAndroid`)

### Audio Capture & Technical Restrictions:
- **Microphone Capture**: Captures local microphone audio via Android `MediaRecorder` using AAC compression in MPEG-4 container (`.m4a`).
- **Android OS AudioPlaybackCapture Restrictions**:
  - Android 10+ (API 29+) introduced `AudioPlaybackCaptureConfiguration`.
  - However, Android strictly restricts playback capture based on the source app's `AudioAttributes`.
  - Google Meet and VoIP telephony applications utilize `AudioAttributes.USAGE_VOICE_COMMUNICATION`, which defaults to `ALLOW_CAPTURE_BY_SYSTEM` or `ALLOW_CAPTURE_BY_NONE` to protect privacy and prevent third-party eavesdropping.
  - **Honest Platform Reality**: Third-party mobile apps cannot intercept remote participants' voices without system rooting.
  - **Classification**: Android recordings are explicitly classified and displayed in the UI as **"Microphone Only (1-Sided Recording)"**.

### Storage & Safety:
- **Output Directory**: `context.getExternalFilesDir(Environment.DIRECTORY_RECORDINGS)`.
- **Database Catalog**: Persisted in Room database `recordings` table.
- **Permission**: Requires `android.permission.RECORD_AUDIO`.
