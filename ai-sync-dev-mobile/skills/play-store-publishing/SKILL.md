---
name: play-store-publishing
description: >-
  Publish Android apps to the Google Play Store using Fastlane Supply. Handles
  authentication setup, AAB/APK uploads, track management (internal/alpha/beta/production),
  staged rollouts, metadata and screenshot management, track promotion, and release validation.
  Use when the user wants to release to Google Play, manage Android tracks, upload APK/AAB,
  or manage Play Store metadata.
allowed-tools: Bash(fastlane:*), Bash(which:fastlane), Bash(brew:*), Bash(gem:*)
triggers:
  - play store
  - google play
  - fastlane supply
  - android release
  - play console
  - upload aab
  - upload apk
---

# Play Store Publishing

Publish Android apps using `fastlane supply` - works as a standalone CLI without a Fastfile.

**Reference files** (read as needed):
- `references/supply-flag-reference.md` - Complete flag/option cheatsheet with env vars and metadata structure
- `references/workflows.md` - End-to-end workflow examples (staged rollout, metadata, CI/CD, ProGuard, multi-locale)

## Setup

```bash
# Check installation
which fastlane

# Install
brew install fastlane
# Or: gem install fastlane

# Verify
fastlane --version
```

## Authentication

Requires a Google Play Developer API service account JSON key.

### Create Service Account

1. Go to **Google Cloud Console** > IAM & Admin > Service Accounts
2. Create service account, download JSON key file
3. In **Play Console** > Settings > API access > Link the service account
4. Grant **"Release Manager"** permissions

### Verify Authentication

```bash
fastlane supply init \
  --json_key path/to/service-account.json \
  --package_name com.example.app
```

This downloads existing metadata and confirms credentials work.

**Env var alternative:** `export SUPPLY_JSON_KEY=path/to/service-account.json`

## Core Workflows

All commands use `fastlane supply` directly (no Fastfile needed). Always include `--json_key` or set `SUPPLY_JSON_KEY`.

### Upload AAB to Internal Track

```bash
fastlane supply \
  --aab path/to/app.aab \
  --track internal \
  --json_key service-account.json \
  --package_name com.example.app
```

### Upload AAB to Production

```bash
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --json_key service-account.json \
  --package_name com.example.app
```

### Upload APK

```bash
fastlane supply \
  --apk path/to/app.apk \
  --track beta \
  --json_key service-account.json \
  --package_name com.example.app
```

### Validate Only (Dry Run)

```bash
fastlane supply \
  --aab path/to/app.aab \
  --track beta \
  --validate_only \
  --json_key service-account.json \
  --package_name com.example.app
```

### Promote Between Tracks

```bash
fastlane supply \
  --track beta \
  --track_promote_to production \
  --json_key service-account.json \
  --package_name com.example.app
```

**Note:** Only the most recent version on a track can be promoted.

## Staged Rollout

### Start Staged Rollout (10%)

```bash
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --rollout 0.1 \
  --release_status inProgress \
  --json_key service-account.json \
  --package_name com.example.app
```

### Increase Rollout (50%)

```bash
fastlane supply \
  --track production \
  --rollout 0.5 \
  --release_status inProgress \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app
```

### Complete Rollout (100%)

```bash
fastlane supply \
  --track production \
  --rollout 1.0 \
  --release_status completed \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app
```

### Halt a Release

```bash
fastlane supply \
  --track production \
  --release_status halted \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app
```

### Recommended Staged Release Flow

1. Upload to **internal** track, validate
2. Promote internal -> **alpha** -> **beta** (monitor 24-48hr at each stage)
3. Promote beta -> **production** at 10% rollout
4. Increase: 10% -> 25% -> 50% -> 100%
5. Halt immediately if crash rate spikes

## Metadata Management

### Initialize Metadata from Play Console

```bash
fastlane supply init \
  --json_key service-account.json \
  --package_name com.example.app
```

Creates the metadata directory structure:

```
fastlane/metadata/android/
├── en-US/
│   ├── title.txt              # App title
│   ├── short_description.txt  # 80 chars max
│   ├── full_description.txt   # 4000 chars max
│   ├── video.txt              # YouTube promo URL
│   ├── changelogs/
│   │   ├── default.txt        # Default changelog
│   │   └── 100.txt            # Version code-specific
│   └── images/
│       ├── icon.png           # 512x512
│       ├── featureGraphic.png # 1024x500
│       ├── phoneScreenshots/
│       ├── sevenInchScreenshots/
│       ├── tenInchScreenshots/
│       ├── tvScreenshots/
│       └── wearScreenshots/
├── de-DE/                     # Additional locales
└── ja-JP/
```

### Upload Metadata Only

```bash
fastlane supply \
  --skip_upload_aab \
  --skip_upload_apk \
  --json_key service-account.json \
  --package_name com.example.app
```

### Screenshot Ordering

Screenshots are ordered alphabetically by filename. Use numeric prefixes:

```
phoneScreenshots/
├── 01_welcome.png
├── 02_dashboard.png
├── 03_settings.png
```

## Release Status Options

| Status | Meaning |
|--------|---------|
| `draft` | Not visible to users, editable |
| `completed` | Full rollout to all users |
| `inProgress` | Staged rollout at specified percentage |
| `halted` | Paused rollout, no new users receive update |

## Available Skip Flags

| Flag | Skips |
|------|-------|
| `--skip_upload_aab` | AAB binary upload |
| `--skip_upload_apk` | APK binary upload |
| `--skip_upload_metadata` | Title, descriptions, etc. |
| `--skip_upload_images` | All image assets |
| `--skip_upload_screenshots` | Screenshot images only |
| `--skip_upload_changelogs` | Changelog files |

**Known issue:** Using more than 2 `--skip*` flags on CLI may be ignored due to a parsing bug. Use a Fastfile for complex skip combinations.

## In-App Update Priority

```bash
fastlane supply \
  --aab app.aab \
  --track production \
  --in_app_update_priority 5 \
  --json_key service-account.json \
  --package_name com.example.app
```

Priority 0 (default) to 5 (highest urgency). Affects Android in-app update API behavior.

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|---------|
| "Only releases with status draft may be created on draft app" | App setup incomplete in Play Console | Complete all setup questionnaires in Play Console UI first |
| "Only one completed release is allowed" | `release_status` defaults to `completed` | Add `--release_status inProgress` for staged rollouts |
| Multiple `--skip*` flags ignored | CLI parsing bug with >2 skip args | Use a Fastfile instead of CLI flags |
| Upload succeeds but nothing changes | Binary path not specified | Verify AAB/APK path in command output |
| Health features declaration error | Google requires explicit declaration | Complete declaration in Play Console manually |
| Track promotion fails | Promoting non-latest version | Only the most recent version on a track can be promoted |
| Permission denied | Service account lacks permissions | Grant "Release Manager" role in Play Console > API access |
| "Package not found" | Wrong package name or not linked | Verify `--package_name` matches Play Console app ID |
