# Store Publishing Skills - Research Report & Proposal

## Executive Summary

This report evaluates CLI tools for automating App Store and Play Store publishing, and proposes two ai-sync skills for the `ai-sync-dev-mobile` package:

1. **`app-store-publishing`** - wrapping [App Store Connect CLI (`asc`)](https://github.com/rudrankriyam/App-Store-Connect-CLI) by Rudrank Riyam
2. **`play-store-publishing`** - wrapping [Fastlane Supply (`fastlane supply`)](https://docs.fastlane.tools/actions/supply/) by the Fastlane team

---

## Part 1: Tool Research

### App Store Connect CLI (`asc`)

| Attribute | Details |
|-----------|---------|
| **Repository** | [rudrankriyam/App-Store-Connect-CLI](https://github.com/rudrankriyam/App-Store-Connect-CLI) |
| **Language** | Go (single binary) |
| **Stars** | ~569 |
| **License** | MIT |
| **Latest Version** | 0.24.x (Feb 2026) |
| **Activity** | Very active - multiple releases per week |
| **Install** | `brew tap rudrankriyam/tap && brew install rudrankriyam/tap/asc` |

#### Why This Tool

- **Single Go binary** - instant startup, no runtime dependencies
- **JSON output by default** - perfect for AI agent parsing and scripting
- **No interactive prompts** - all flag-based, CI/CD-ready
- **Comprehensive coverage** - builds, versions, TestFlight, reviews, analytics, Game Center, Xcode Cloud
- **Multiple output formats** - JSON, table, markdown
- **Active development** - rapid iteration with weekly releases

#### Key Capabilities

| Category | Commands |
|----------|----------|
| **Apps** | `apps list`, `apps get` |
| **Versions** | `versions list/get/create`, `versions attach-build` |
| **Builds** | `builds upload --ipa`, `builds list/get`, `builds add-groups` |
| **Submission** | `submit create --confirm`, `submit status` |
| **TestFlight** | `beta-groups list/create`, `beta-testers add/invite`, `feedback`, `crashes` |
| **Reviews** | `reviews list --stars N`, `reviews respond` |
| **Analytics** | `analytics sales`, `analytics request` |
| **Devices** | `devices list/register/update` |
| **Auth** | `auth login/switch`, multi-profile support, keychain storage |

#### Authentication Model

```bash
# Register API credentials (from App Store Connect > Users and Access > Integrations > API)
asc auth login \
  --name "MyProfile" \
  --key-id "ABC123" \
  --issuer-id "DEF456" \
  --private-key /path/to/AuthKey.p8

# Credential hierarchy: Keychain > ./.asc/config.json > ~/.asc/config.json > env vars
# Key env vars: ASC_KEY_ID, ASC_ISSUER_ID, ASC_PRIVATE_KEY_PATH, ASC_APP_ID
```

#### Known Limitations

- **Build upload**: Prepares upload operations but full automation (upload + commit) still maturing
- **Metadata management**: Limited compared to web interface for screenshots/previews
- **Some web-only operations**: App version creation, encryption compliance, IAP configuration
- **Rate limits**: Apple enforces primary and secondary rate limits; secondary limits may require 1hr wait
- **JWT expiry**: Tokens expire after 10 minutes (CLI handles renewal automatically)
- **POST/PATCH/DELETE**: Not automatically retried on 429/503 (only GET/HEAD are retried)

---

### Fastlane Supply (Play Store)

| Attribute | Details |
|-----------|---------|
| **Repository** | [fastlane/fastlane](https://github.com/fastlane/fastlane) |
| **Component** | `supply` / `upload_to_play_store` action |
| **Language** | Ruby |
| **Stars** | ~40.4k |
| **License** | MIT |
| **Latest Version** | 2.228.0 (June 2025) |
| **Activity** | Actively maintained, 559 releases over 10 years |
| **Install** | `brew install fastlane` or `gem install fastlane` |

#### Why This Tool (Over Alternatives)

| Tool | Verdict |
|------|---------|
| **Fastlane Supply** | **Winner** - most feature-complete, battle-tested, works as standalone CLI |
| google-play-cli (Vacxe) | Runner-up - standalone binary but lacks metadata/screenshots/rollout support |
| Gradle Play Publisher | Gradle plugin only, requires project context |
| apkup (eventOneHQ) | Abandoned (2021), no AAB support |
| playup / playup3 | Deprecated |
| google-playstore-publisher | Abandoned, no AAB support |

Fastlane Supply wins because:
- **Works as pure CLI** without Fastfile: `fastlane supply --aab app.aab --track beta --json_key creds.json`
- **Full feature coverage**: APK/AAB upload, track management, promotion, staged rollout, metadata, screenshots, changelogs, validation
- **Massive community**: 40k+ stars, 1,550+ contributors, production-proven
- **Simple install**: `brew install fastlane`

#### Key Capabilities

| Feature | Flag/Usage |
|---------|-----------|
| **Upload AAB** | `--aab path/app.aab` |
| **Upload APK** | `--apk path/app.apk` |
| **Track selection** | `--track internal/alpha/beta/production` |
| **Track promotion** | `--track beta --track_promote_to production` |
| **Staged rollout** | `--rollout 0.1` (0.0 to 1.0) + `--release_status inProgress` |
| **Release status** | `--release_status draft/completed/halted/inProgress` |
| **Metadata** | `--metadata_path ./metadata/android` |
| **Validate only** | `--validate_only` |
| **Init metadata** | `fastlane supply init --json_key creds.json --package_name com.example` |
| **Skip options** | `--skip_upload_apk/aab/metadata/images/screenshots/changelogs` |
| **Update priority** | `--in_app_update_priority 0-5` |

#### Authentication Model

```bash
# Requires Google Play Developer API service account JSON
# 1. Go to Google Cloud Console > Create Service Account
# 2. Download JSON key file
# 3. In Play Console > Settings > API access > Link service account
# 4. Grant "Release Manager" permissions

fastlane supply --json_key path/to/service-account.json ...
# OR via env var:
export SUPPLY_JSON_KEY=path/to/service-account.json
```

#### Metadata Directory Structure

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
├── de-DE/
│   └── ... (same structure)
└── ja-JP/
    └── ... (same structure)
```

#### Known Limitations & Pitfalls

| Issue | Cause | Solution |
|-------|-------|---------|
| "Only releases with status draft may be created on draft app" | App setup incomplete in Play Console | Complete all setup questionnaires in Play Console UI first |
| "Only one completed release is allowed" | Fastlane 2.226.0+ defaults `release_status` to `completed` | Add `--release_status inProgress` for partial rollouts |
| Multiple `--skip*` flags ignored | CLI parsing bug with >2 skip args | Use Fastfile instead of CLI flags |
| Upload succeeds but nothing changes | AAB/APK path not specified | Always verify binary path in summary output |
| Health features declaration error | Google requires explicit health feature declaration | Complete declaration in Play Console manually |
| Track promotion fails | Trying to promote non-latest version | Only the most recent version on a track can be promoted |

---

## Part 2: Skill Proposals

### Skill 1: `app-store-publishing`

```
skills/
└── app-store-publishing/
    ├── SKILL.md                     # Main skill file
    └── references/
        ├── asc-command-reference.md # Complete command cheatsheet
        └── workflows.md            # End-to-end workflow examples
```

#### SKILL.md Content Outline

```yaml
---
name: app-store-publishing
description: >-
  Publish iOS/macOS apps to the App Store using the asc CLI. Handles
  authentication setup, build uploads, TestFlight distribution, version
  management, app submission for review, and review status checking.
  Trigger for: app store release, submit to apple, testflight, asc commands.
triggers:
  - app store
  - submit to apple
  - testflight
  - asc
  - ios release
  - app store connect
---
```

**Sections:**

1. **Setup & Prerequisites**
   - Check if `asc` is installed: `which asc`
   - Install: `brew tap rudrankriyam/tap && brew install rudrankriyam/tap/asc`
   - Update: `brew upgrade rudrankriyam/tap/asc`
   - Verify: `asc --version`

2. **Authentication Setup**
   - Guide user to App Store Connect > Users & Access > Integrations > API
   - `asc auth login` with key-id, issuer-id, private-key path
   - Verify: `asc auth status` or `asc apps list`
   - Multi-profile support for multiple teams

3. **Core Workflows**
   - **Upload Build**: `asc builds upload --app APP_ID --ipa path/to/app.ipa`
   - **Check Build Status**: `asc builds list --app APP_ID --sort "-uploadedDate"`
   - **Create Version**: `asc versions create --app APP_ID --version "1.0.0" --platform iOS`
   - **Attach Build to Version**: `asc versions attach-build --version-id VER_ID --build BUILD_ID`
   - **Submit for Review**: `asc submit create --app APP_ID --version "1.0.0" --build BUILD_ID --confirm`
   - **Check Submission Status**: `asc submit status --version-id VER_ID`

4. **TestFlight Workflow**
   - List beta groups: `asc beta-groups list --app APP_ID`
   - Add build to group: `asc builds add-groups --build BUILD_ID --group GROUP_ID --notify --wait`
   - Add testers: `asc beta-testers add --app APP_ID --email EMAIL --group GROUP`
   - Check feedback: `asc feedback --app APP_ID`
   - View crashes: `asc crashes --app APP_ID`

5. **Monitoring & Reviews**
   - App reviews: `asc reviews list --app APP_ID --paginate`
   - Respond to reviews: `asc reviews respond --review-id ID --response "Thank you!"`
   - Sales data: `asc analytics sales --vendor VENDOR_ID --type SALES --date DATE`

6. **Troubleshooting**
   - Common error patterns and fixes
   - Rate limit handling
   - Build validation failures

---

### Skill 2: `play-store-publishing`

```
skills/
└── play-store-publishing/
    ├── SKILL.md                       # Main skill file
    └── references/
        ├── supply-flag-reference.md   # Complete flag cheatsheet
        ├── metadata-structure.md      # Directory structure guide
        └── workflows.md              # End-to-end workflow examples
```

#### SKILL.md Content Outline

```yaml
---
name: play-store-publishing
description: >-
  Publish Android apps to the Google Play Store using Fastlane Supply.
  Handles authentication setup, AAB/APK uploads, track management,
  staged rollouts, metadata/screenshot management, and release promotion.
  Trigger for: play store release, google play, fastlane supply, android release.
triggers:
  - play store
  - google play
  - fastlane supply
  - android release
  - play console
---
```

**Sections:**

1. **Setup & Prerequisites**
   - Check if `fastlane` is installed: `which fastlane`
   - Install: `brew install fastlane` or `gem install fastlane`
   - Verify: `fastlane --version`
   - Note: Ruby runtime required (Homebrew handles transparently)

2. **Authentication Setup**
   - Guide to Google Cloud Console > Service Account creation
   - Download JSON key file
   - Link in Play Console > Settings > API access
   - Grant "Release Manager" permissions
   - Verify: `fastlane supply init --json_key creds.json --package_name com.example.app`

3. **Core Workflows**
   - **Upload AAB to Internal**: `fastlane supply --aab app.aab --track internal --json_key creds.json`
   - **Upload AAB to Production**: `fastlane supply --aab app.aab --track production --json_key creds.json`
   - **Staged Rollout**: `fastlane supply --aab app.aab --track production --rollout 0.1 --release_status inProgress --json_key creds.json`
   - **Promote Track**: `fastlane supply --track beta --track_promote_to production --json_key creds.json`
   - **Increase Rollout**: `fastlane supply --track production --rollout 0.5 --release_status inProgress --skip_upload_aab --skip_upload_metadata --json_key creds.json`
   - **Complete Rollout**: `fastlane supply --track production --rollout 1.0 --release_status completed --skip_upload_aab --skip_upload_metadata --json_key creds.json`
   - **Validate Only**: `fastlane supply --aab app.aab --track beta --validate_only --json_key creds.json`

4. **Metadata Management**
   - Initialize from Play Console: `fastlane supply init`
   - Directory structure explanation
   - Updating titles, descriptions, changelogs per locale
   - Screenshot management (naming convention for ordering)
   - Upload metadata only: `fastlane supply --skip_upload_aab --json_key creds.json`

5. **Staged Release Workflow** (step-by-step)
   - Upload to internal (10% validation)
   - Promote internal -> alpha -> beta -> production
   - Monitor at each stage (24-48hr)
   - Increase rollout: 10% -> 25% -> 50% -> 100%
   - Emergency halt guidance

6. **Troubleshooting**
   - Common errors with solutions table
   - Draft app setup requirements
   - Health feature declaration
   - Skip flag parsing workaround
   - Service account permission issues

---

## Part 3: Comparison Summary

| Aspect | App Store (asc) | Play Store (fastlane supply) |
|--------|----------------|------------------------------|
| **Binary** | Go single binary | Ruby gem |
| **Install** | `brew install asc` | `brew install fastlane` |
| **Auth** | API Key + .p8 file | Service Account JSON |
| **Output** | JSON/table/markdown | Text (Fastlane format) |
| **Build Upload** | `asc builds upload --ipa` | `fastlane supply --aab` |
| **Track/TestFlight** | `asc builds add-groups` | `--track internal/alpha/beta/production` |
| **Staged Rollout** | N/A (Apple manages) | `--rollout 0.1 --release_status inProgress` |
| **Metadata** | Partial CLI support | Full directory-based system |
| **Submit for Review** | `asc submit create --confirm` | N/A (Play Store auto-publishes) |
| **Review Status** | `asc submit status` | N/A (check Play Console) |
| **Maturity** | Growing (0.24.x) | Battle-tested (2.228.0) |

---

## Part 4: Recommendation

### Proceed with Both Skills

Both tools are the clear best choices for their respective platforms:

- **`asc`** is the only viable standalone CLI for App Store Connect (alternatives are either part of Fastlane's Ruby ecosystem or less feature-complete)
- **`fastlane supply`** is the undisputed leader for Play Store CLI publishing (no alternative comes close in features)

### Suggested Implementation Order

1. **`app-store-publishing`** first - the `asc` CLI is more self-contained and the workflow is more linear (upload -> attach -> submit -> monitor)
2. **`play-store-publishing`** second - slightly more complex due to metadata directory structure and staged rollout patterns

### Skill Complexity Estimate

Both skills should use the **complex skill pattern** (directory with `SKILL.md` + `references/`):
- Main `SKILL.md` stays under 500 lines with setup, core workflows, and troubleshooting
- Reference files contain full command cheatsheets and extended workflow examples
- No scripts needed (both CLIs are invoked directly via Bash)

---

## Sources

- [App Store Connect CLI - GitHub](https://github.com/rudrankriyam/App-Store-Connect-CLI)
- [App Store Connect CLI - Releases](https://github.com/rudrankriyam/App-Store-Connect-CLI/releases)
- [Fastlane Supply Documentation](https://docs.fastlane.tools/actions/supply/)
- [Fastlane upload_to_play_store](https://docs.fastlane.tools/actions/upload_to_play_store/)
- [google-play-cli (Vacxe)](https://github.com/Vacxe/google-play-cli)
- [Gradle Play Publisher](https://github.com/Triple-T/gradle-play-publisher)
- [Google Play Developer API](https://developers.google.com/android-publisher)
- [Apple App Store Connect API](https://developer.apple.com/documentation/appstoreconnectapi)
