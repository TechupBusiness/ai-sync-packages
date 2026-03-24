# ASC CLI - Complete Command Reference

Full command cheatsheet for the `asc` CLI (App Store Connect CLI).

## Global Flags

| Flag | Purpose |
|------|---------|
| `--profile "Name"` | Use specific auth profile |
| `--output json\|table\|markdown` | Output format (default: JSON) |
| `--paginate` | Fetch all pages automatically |
| `--limit N` | Results per page |
| `--sort field\|-field` | Sort by field (- for descending) |
| `--pretty` | Pretty-print JSON |
| `--debug` | Enable debug logging |
| `--api-debug` | Show HTTP requests/responses |
| `--confirm` | Skip confirmation prompts |
| `--dry-run` | Preview without executing |

## Authentication

```bash
# Register credentials
asc auth login --name "Profile" --key-id "ID" --issuer-id "ID" --private-key /path/to/key.p8
asc auth login --network                  # Validate via network
asc auth login --skip-validation          # Skip JWT validation (CI-friendly)

# Config templates
asc auth init                             # Create template config
asc auth init --local                     # Repo-local config (./.asc/config.json)

# Profile management
asc auth switch --name "ProfileName"
asc auth status
asc auth status --verbose
asc auth status --validate

# Diagnostics
asc auth doctor
asc auth doctor --fix --confirm
asc auth logout
```

## Apps

```bash
asc apps list [--sort name|bundleId] [--paginate]
asc apps get --app "APP_ID"
```

## Builds

```bash
asc builds list --app "APP_ID" [--sort "-uploadedDate"] [--paginate]
asc builds info --build "BUILD_ID"
asc builds upload --app "APP_ID" --ipa "file.ipa"
asc builds expire --build "BUILD_ID"
asc builds expire-all --app "APP_ID" --older-than 90d [--dry-run] [--confirm]
asc builds add-groups --build "BUILD_ID" --group "GROUP_ID" [--notify] [--wait]
asc builds remove-groups --build "BUILD_ID" --group "GROUP_ID"
```

## Versions

```bash
asc versions list --app "APP_ID" [--paginate]
asc versions get --version-id "VERSION_ID"
asc versions create --app "APP_ID" --version "1.0.0" --platform iOS
asc versions attach-build --version-id "VERSION_ID" --build "BUILD_ID"
asc versions release --version-id "VERSION_ID" --confirm
```

### Phased Releases

```bash
asc versions phased-release get --version-id "VERSION_ID"
asc versions phased-release create --version-id "VERSION_ID"
asc versions phased-release update --id "PHASED_ID" --state PAUSED
```

### Version Promotions

```bash
asc versions promotions create --version-id "VERSION_ID" --treatment-id "TREATMENT_ID"
```

## Pre-Release Versions

```bash
asc pre-release-versions list --app "APP_ID" [--platform] [--version] [--paginate]
asc pre-release-versions get --id "ID"
```

## Submissions

```bash
asc submit create --app "APP_ID" --version "1.0.0" --build "BUILD_ID" --confirm
asc submit status --id "SUBMISSION_ID"
asc submit status --version-id "VERSION_ID"
asc submit cancel --id "SUBMISSION_ID" --confirm
```

## TestFlight - Beta Groups

```bash
asc beta-groups list --app "APP_ID" [--paginate]
asc beta-groups create --app "APP_ID" --name "Name"
asc beta-groups get --id "GROUP_ID"
asc beta-groups update --id "GROUP_ID" --name "NewName"
asc beta-groups delete --id "GROUP_ID" --confirm
asc beta-groups add-testers --group "GROUP_ID" --tester "TESTER_ID"
asc beta-groups remove-testers --group "GROUP_ID" --tester "TESTER_ID"
```

## TestFlight - Beta Testers

```bash
asc beta-testers list --app "APP_ID" [--build BUILD_ID] [--group GROUP_ID] [--paginate]
asc beta-testers get --id "TESTER_ID"
asc beta-testers add --app "APP_ID" --email "email" --group "GroupName"
asc beta-testers remove --app "APP_ID" --email "email"
asc beta-testers invite --app "APP_ID" --email "email"
asc beta-testers add-groups --id "TESTER_ID" --group "GROUP_ID"
asc beta-testers remove-groups --id "TESTER_ID" --group "GROUP_ID"
```

## TestFlight - App Management

```bash
asc testflight apps list
asc testflight apps get --app "APP_ID"
asc testflight sync pull --app "APP_ID" --output "./file.yaml"
```

## Feedback & Crashes

```bash
asc feedback --app "APP_ID" [--device-model] [--os-version] [--paginate]
asc crashes --app "APP_ID" [--output table|markdown] [--sort] [--limit]
```

## Reviews

```bash
asc reviews list --app "APP_ID" [--stars 1-5] [--territory] [--sort] [--output] [--paginate]
asc reviews respond --review-id "REVIEW_ID" --response "Text"
asc reviews response get --id "RESPONSE_ID"
asc reviews response for-review --review-id "REVIEW_ID"
asc reviews response delete --id "RESPONSE_ID" --confirm
```

## Devices

```bash
asc devices list [--platform IOS] [--status ENABLED] [--udid] [--paginate]
asc devices get --id "DEVICE_ID"
asc devices register --name "Name" --udid "UDID" --platform IOS
asc devices update --id "DEVICE_ID" --name "NewName"
asc devices update --id "DEVICE_ID" --status DISABLED
```

## Localizations

```bash
# Version localizations
asc localizations list --version "VERSION_ID" [--paginate]
asc localizations list --app "APP_ID" --type app-info [--paginate]
asc localizations download --version "VERSION_ID" --path "./dir"
asc localizations upload --version "VERSION_ID" --path "./dir"

# Build localizations (What's New in This Version)
asc build-localizations list --build "BUILD_ID" [--paginate]
asc build-localizations create --build "BUILD_ID" --locale "en-US" --whats-new "Text"
asc build-localizations update --id "LOC_ID" --whats-new "Text"
asc build-localizations delete --id "LOC_ID" --confirm
```

## App Setup & Metadata

```bash
# App info
asc app-setup info set --app "APP_ID" --primary-locale "en-US" --bundle-id "com.example"
asc app-setup info set --app "APP_ID" --locale "en-US" --name "Name" --subtitle "Subtitle"
asc app-info get --app "APP_ID" [--version] [--platform]
asc app-info set --app "APP_ID" --locale "en-US" --whats-new "Text"

# Categories
asc categories list [--output table]
asc categories set --app "APP_ID" --primary GAMES
asc app-setup categories set --app "APP_ID" --primary GAMES [--secondary TYPE]

# Availability & Pricing
asc app-setup availability set --app "APP_ID" --territory "USA" --available true
asc app-setup pricing set --app "APP_ID" --price-point "ID" --base-territory "USA"
```

## App Tags

```bash
asc app-tags list --app "APP_ID" [--visible-in-app-store] [--include territories]
asc app-tags get --app "APP_ID" --id "TAG_ID"
asc app-tags update --id "TAG_ID" --visible-in-app-store=false --confirm
asc app-tags territories --id "TAG_ID"
asc app-tags relationships --app "APP_ID"
```

## App Events

```bash
asc app-events list --app "APP_ID"
asc app-events localizations list --event-id "EVENT_ID"
asc app-events localizations screenshots list --localization-id "LOC_ID"
asc app-events localizations video-clips list --localization-id "LOC_ID"
```

## Analytics & Financial Reports

```bash
# Sales reports
asc analytics sales --vendor "VENDOR_ID" --type SALES --subtype SUMMARY \
  --frequency DAILY --date "YYYY-MM-DD" [--decompress]

# Analytics requests
asc analytics request --app "APP_ID" --access-type ONGOING
asc analytics requests --app "APP_ID" [--paginate]
asc analytics get --request-id "REQ_ID" [--date] [--include-segments]
asc analytics download --request-id "REQ_ID" --instance-id "INSTANCE_ID"

# Financial reports
asc finance reports --vendor "VENDOR_ID" --report-type FINANCIAL --region "ZZ" \
  --date "YYYY-MM" [--decompress]
asc finance reports --vendor "VENDOR_ID" --report-type FINANCE_DETAIL --region "Z1" \
  --date "YYYY-MM"
asc finance regions [--output table]
```

## Game Center

### Achievements

```bash
asc game-center achievements list --app "APP_ID"
asc game-center achievements create --app "APP_ID" --reference-name "Name" \
  --vendor-id "id" --points 10
asc game-center achievements update --id "ACH_ID" --points 20
asc game-center achievements delete --id "ACH_ID" --confirm
asc game-center achievements localizations list --achievement-id "ACH_ID"
asc game-center achievements images upload --localization-id "LOC_ID" --file "path"
asc game-center achievements releases list --achievement-id "ACH_ID"
asc game-center achievements releases create --app "APP_ID" --achievement-id "ACH_ID"
```

### Leaderboards

```bash
asc game-center leaderboards list --app "APP_ID"
asc game-center leaderboards create --app "APP_ID" --reference-name "Name" \
  --vendor-id "id" --formatter INTEGER --sort DESC
asc game-center leaderboards localizations list --leaderboard-id "LB_ID"
asc game-center leaderboards images upload --localization-id "LOC_ID" --file "path"
asc game-center leaderboards releases list --leaderboard-id "LB_ID"
```

### Leaderboard Sets

```bash
asc game-center leaderboard-sets list --app "APP_ID"
asc game-center leaderboard-sets create --app "APP_ID" --reference-name "Name"
asc game-center leaderboard-sets members list --set-id "SET_ID"
asc game-center leaderboard-sets members set --set-id "SET_ID" --leaderboard-ids "id1,id2"
asc game-center leaderboard-sets localizations list --set-id "SET_ID"
asc game-center leaderboard-sets images upload --localization-id "LOC_ID" --file "path"
```

## Sandbox Testing

```bash
asc sandbox list [--email] [--territory] [--paginate]
asc sandbox get --id "SANDBOX_ID"
asc sandbox get --email "email"
asc sandbox update --id "SANDBOX_ID" --territory "USA"
asc sandbox update --id "SANDBOX_ID" \
  --subscription-renewal-rate "MONTHLY_RENEWAL_EVERY_ONE_HOUR"
asc sandbox clear-history --id "SANDBOX_ID" --confirm
```

## Xcode Cloud

```bash
asc xcode-cloud workflows --app "APP_ID" [--paginate]
asc xcode-cloud build-runs --workflow-id "WORKFLOW_ID" [--paginate]
asc xcode-cloud run --app "APP_ID" --workflow "Name" --branch "main" [--wait]
asc xcode-cloud run --workflow-id "WORKFLOW_ID" --git-reference-id "REF_ID" [--wait]
asc xcode-cloud status --run-id "RUN_ID" [--wait] [--output]
```

## Offer Codes

```bash
asc offer-codes list --offer-code "OFFER_ID" [--paginate]
asc offer-codes generate --offer-code "OFFER_ID" --quantity 10 \
  --expiration-date "2026-12-31"
asc offer-codes values --id "BATCH_ID" --output "./codes.txt"
```

## Alternative Distribution (EU)

```bash
asc alternative-distribution domains list
asc alternative-distribution domains create --domain "example.com" \
  --reference-name "Name"
asc alternative-distribution keys create --app "APP_ID" \
  --public-key-path "./key.pem"
asc alternative-distribution keys app --app "APP_ID"
asc alternative-distribution packages create --app-store-version-id "VERSION_ID"
asc alternative-distribution packages versions list --package-id "PKG_ID"
asc alternative-distribution packages versions variants --version-id "VERSION_ID"
```

## Migration (from Fastlane)

```bash
asc migrate validate --fastlane-dir ./metadata
asc migrate import --app "APP_ID" --fastlane-dir ./metadata
asc migrate export --app "APP_ID" --output ./metadata
```

## Environment Variables

### Authentication

| Variable | Purpose |
|----------|---------|
| `ASC_KEY_ID` | API key identifier |
| `ASC_ISSUER_ID` | Issuer identifier |
| `ASC_PRIVATE_KEY_PATH` | Path to .p8 private key |
| `ASC_PRIVATE_KEY` | Raw private key content |
| `ASC_PRIVATE_KEY_B64` | Base64-encoded private key |
| `ASC_CONFIG_PATH` | Config file path override |
| `ASC_PROFILE` | Default profile name |
| `ASC_BYPASS_KEYCHAIN` | Skip keychain, use config file |

### Timeouts & Retry

| Variable | Default | Purpose |
|----------|---------|---------|
| `ASC_TIMEOUT` / `ASC_TIMEOUT_SECONDS` | - | General request timeout |
| `ASC_UPLOAD_TIMEOUT` / `ASC_UPLOAD_TIMEOUT_SECONDS` | - | Upload timeout |
| `ASC_MAX_RETRIES` | 3 | Max retry attempts |
| `ASC_BASE_DELAY` | 1s | Initial retry delay |
| `ASC_MAX_DELAY` | 30s | Max retry delay |
| `ASC_RETRY_LOG` | - | Set to `1` to log retries |

### Convenience

| Variable | Purpose |
|----------|---------|
| `ASC_APP_ID` | Default app ID |
| `ASC_VENDOR_NUMBER` | Default vendor number |
| `ASC_ANALYTICS_VENDOR_NUMBER` | Analytics vendor override |
| `ASC_DEBUG` | Set to `1` or `api` for debug output |
| `ASC_NO_UPDATE` | Set to `1` to disable update checks |
