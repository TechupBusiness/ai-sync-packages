---
name: app-store-publishing
description: >-
  Publish iOS/macOS apps to the App Store using the asc CLI (App Store Connect CLI).
  Handles authentication setup, build uploads, TestFlight distribution, version management,
  app submission for review, and review monitoring. Use when the user wants to release to
  the App Store, manage TestFlight, upload builds, submit for Apple review, or check review status.
allowed-tools: Bash(asc:*), Bash(which:asc), Bash(brew:*)
triggers:
  - app store
  - submit to apple
  - testflight
  - asc
  - ios release
  - app store connect
  - apple review
---

# App Store Publishing

Publish iOS/macOS apps using the `asc` CLI (App Store Connect CLI) - a Go binary with JSON output, no interactive prompts.

**Reference files** (read as needed):
- `references/asc-command-reference.md` - Complete command cheatsheet with all flags, subcommands, and env vars
- `references/workflows.md` - End-to-end workflow examples (TestFlight, phased release, Xcode Cloud, Game Center, analytics)

## Setup

```bash
# Check installation
which asc

# Install
brew tap rudrankriyam/tap && brew install rudrankriyam/tap/asc

# Update
brew upgrade rudrankriyam/tap/asc

# Verify
asc --version
```

## Authentication

Requires an App Store Connect API key from: App Store Connect > Users and Access > Integrations > Individual Keys.

```bash
# Register credentials (stored in keychain)
asc auth login \
  --name "MyProfile" \
  --key-id "YOUR_KEY_ID" \
  --issuer-id "YOUR_ISSUER_ID" \
  --private-key /path/to/AuthKey_KEYID.p8

# Verify authentication
asc apps list

# Switch between profiles (multi-team)
asc auth switch --name "OtherProfile"
```

**Credential resolution order:** Keychain > `./.asc/config.json` > `~/.asc/config.json` > env vars (`ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_PRIVATE_KEY_PATH`, `ASC_APP_ID`).

## Core Workflows

### Upload a Build

```bash
asc builds upload --app APP_ID --ipa path/to/app.ipa
```

### Check Build Status

```bash
# Latest builds (JSON output)
asc builds list --app APP_ID --sort "-uploadedDate"

# Specific build
asc builds get --build BUILD_ID
```

### Create a New Version

```bash
asc versions create --app APP_ID --version "1.2.0" --platform iOS
```

### Attach Build to Version

```bash
asc versions attach-build --version-id VERSION_ID --build BUILD_ID
```

### Submit for Review

```bash
asc submit create --app APP_ID --version "1.2.0" --build BUILD_ID --confirm
```

### Check Submission Status

```bash
asc submit status --version-id VERSION_ID
```

### Full Release Flow

```bash
# 1. Upload build
asc builds upload --app APP_ID --ipa app.ipa

# 2. Wait for processing, check status
asc builds list --app APP_ID --sort "-uploadedDate"

# 3. Create version
asc versions create --app APP_ID --version "1.2.0" --platform iOS

# 4. Attach build to version
asc versions attach-build --version-id VERSION_ID --build BUILD_ID

# 5. Submit for review
asc submit create --app APP_ID --version "1.2.0" --build BUILD_ID --confirm

# 6. Monitor review status
asc submit status --version-id VERSION_ID
```

## TestFlight

### Distribute to Beta Group

```bash
# List beta groups
asc beta-groups list --app APP_ID

# Add build to group (with notification)
asc builds add-groups --build BUILD_ID --group GROUP_ID --notify --wait

# Add individual tester
asc beta-testers add --app APP_ID --email tester@example.com --group GROUP_NAME

# Invite external tester
asc beta-testers invite --app APP_ID --email external@example.com --group GROUP_NAME
```

### Monitor TestFlight Feedback

```bash
# View feedback
asc feedback --app APP_ID

# View crashes
asc crashes --app APP_ID
```

## Monitoring & Reviews

```bash
# List app reviews (paginated)
asc reviews list --app APP_ID --paginate

# Filter by star rating
asc reviews list --app APP_ID --stars 1

# Respond to a review
asc reviews respond --review-id REVIEW_ID --response "Thank you for your feedback!"

# Sales analytics
asc analytics sales --vendor VENDOR_ID --type SALES --date YYYY-MM-DD
```

## Device Management

```bash
# List registered devices
asc devices list

# Register a new device
asc devices register --name "Test iPhone" --udid DEVICE_UDID --platform iOS
```

## Output Formats

All commands default to JSON. Override with:

```bash
asc builds list --app APP_ID --format table
asc builds list --app APP_ID --format markdown
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|---------|
| Rate limit (429) | Apple API rate limits | Wait and retry; secondary limits may require 1hr wait |
| JWT expired | Tokens expire after 10 min | CLI renews automatically; re-run command |
| POST/PATCH not retried | Only GET/HEAD auto-retry on 429/503 | Retry write operations manually |
| Build processing slow | Apple processing pipeline | Check `asc builds list` periodically; can take 15-30 min |
| Auth failure | Missing or expired API key | Re-run `asc auth login` with valid .p8 key |
| "App not found" | Wrong APP_ID or team | Check `asc apps list` to find correct ID |
