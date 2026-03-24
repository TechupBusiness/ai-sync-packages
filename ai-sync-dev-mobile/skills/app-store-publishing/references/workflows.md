# App Store Publishing Workflows

End-to-end workflow examples for common App Store publishing scenarios.

## Complete App Store Release

Upload a build, create a version, and submit for App Store review.

```bash
# 1. Upload the IPA
asc builds upload --app APP_ID --ipa path/to/MyApp.ipa

# 2. Wait for processing (typically 15-30 min), check status
asc builds list --app APP_ID --sort "-uploadedDate" --limit 1

# 3. Create a new App Store version
asc versions create --app APP_ID --version "2.1.0" --platform iOS

# 4. Attach the processed build to the version
asc versions attach-build --version-id VERSION_ID --build BUILD_ID

# 5. Set localized "What's New" text
asc build-localizations create --build BUILD_ID --locale "en-US" \
  --whats-new "Bug fixes and performance improvements"

# 6. Submit for review
asc submit create --app APP_ID --version "2.1.0" --build BUILD_ID --confirm

# 7. Monitor review status
asc submit status --version-id VERSION_ID
```

## TestFlight Beta Distribution

Distribute a build to internal or external testers via TestFlight.

```bash
# 1. Upload the build
asc builds upload --app APP_ID --ipa path/to/MyApp.ipa

# 2. Check build processing status
asc builds list --app APP_ID --sort "-uploadedDate" --limit 1

# 3. List available beta groups
asc beta-groups list --app APP_ID

# 4. Add build to internal beta group (notifies testers, waits for processing)
asc builds add-groups --build BUILD_ID --group INTERNAL_GROUP_ID --notify --wait

# 5. After internal validation, add to external group
asc builds add-groups --build BUILD_ID --group EXTERNAL_GROUP_ID --notify --wait

# 6. Monitor crash reports and feedback
asc crashes --app APP_ID --sort -createdDate --limit 10
asc feedback --app APP_ID --paginate
```

## Phased Release Management

Roll out a release gradually to App Store users over 7 days.

```bash
# 1. After submitting and getting approved, enable phased release
asc versions phased-release create --version-id VERSION_ID

# 2. Check phased release progress
asc versions phased-release get --version-id VERSION_ID

# 3. Pause if issues detected
asc versions phased-release update --id PHASED_ID --state PAUSED

# 4. Resume when ready
asc versions phased-release update --id PHASED_ID --state ACTIVE
```

**Note:** Apple manages the rollout percentages automatically over 7 days: Day 1: 1%, Day 2: 2%, Day 3: 5%, Day 4: 10%, Day 5: 20%, Day 6: 50%, Day 7: 100%.

## Xcode Cloud CI/CD Integration

Trigger Xcode Cloud builds and monitor their status.

```bash
# 1. List available workflows
asc xcode-cloud workflows --app APP_ID

# 2. Trigger a build and wait for completion
asc xcode-cloud run --app APP_ID --workflow "Release Build" --branch "main" --wait

# 3. Check build status if not using --wait
asc xcode-cloud status --run-id RUN_ID

# 4. List recent build runs for a workflow
asc xcode-cloud build-runs --workflow-id WORKFLOW_ID --paginate
```

## Review Monitoring & Response

Monitor and respond to App Store reviews.

```bash
# View all recent reviews
asc reviews list --app APP_ID --paginate --output table

# Filter negative reviews (1-2 stars)
asc reviews list --app APP_ID --stars 1
asc reviews list --app APP_ID --stars 2

# Filter by territory
asc reviews list --app APP_ID --territory US

# Respond to a review
asc reviews respond --review-id REVIEW_ID \
  --response "Thank you for your feedback! We've addressed this in v2.1."

# Check existing response
asc reviews response for-review --review-id REVIEW_ID

# Delete a response
asc reviews response delete --id RESPONSE_ID --confirm
```

## Analytics & Sales Reporting

Download sales and financial data.

```bash
# Daily sales report
asc analytics sales --vendor VENDOR_ID --type SALES --subtype SUMMARY \
  --frequency DAILY --date "2026-02-01" --decompress

# Monthly financial report
asc finance reports --vendor VENDOR_ID --report-type FINANCIAL \
  --region "US" --date "2026-01"

# Request ongoing analytics access
asc analytics request --app APP_ID --access-type ONGOING

# List analytics requests
asc analytics requests --app APP_ID --paginate

# Download analytics data
asc analytics get --request-id REQ_ID --include-segments
asc analytics download --request-id REQ_ID --instance-id INSTANCE_ID
```

## Device Registration & Management

Register devices for development and ad hoc distribution.

```bash
# List all registered devices
asc devices list --paginate --output table

# Register a new device
asc devices register --name "QA iPhone 15" --udid "DEVICE_UDID" --platform iOS

# Filter by platform
asc devices list --platform IOS --status ENABLED

# Disable a device
asc devices update --id DEVICE_ID --status DISABLED
```

## Sandbox Testing (IAP & Subscriptions)

Manage sandbox tester accounts for in-app purchase testing.

```bash
# List sandbox testers
asc sandbox list --paginate

# Get specific tester by email
asc sandbox get --email "tester@example.com"

# Change territory for testing regional pricing
asc sandbox update --id SANDBOX_ID --territory "JPN"

# Speed up subscription renewals for testing
asc sandbox update --id SANDBOX_ID \
  --subscription-renewal-rate "MONTHLY_RENEWAL_EVERY_ONE_HOUR"

# Clear purchase history (reset for clean testing)
asc sandbox clear-history --id SANDBOX_ID --confirm
```

## Game Center Setup

Configure achievements and leaderboards.

```bash
# Create an achievement
asc game-center achievements create --app APP_ID \
  --reference-name "First Victory" --vendor-id "first_victory" --points 10

# Create a leaderboard
asc game-center leaderboards create --app APP_ID \
  --reference-name "High Score" --vendor-id "high_score" \
  --formatter INTEGER --sort DESC

# Create a leaderboard set (group of leaderboards)
asc game-center leaderboard-sets create --app APP_ID \
  --reference-name "All Scores"

# Add leaderboards to set
asc game-center leaderboard-sets members set --set-id SET_ID \
  --leaderboard-ids "lb1,lb2,lb3"

# Upload localized achievement image
asc game-center achievements images upload \
  --localization-id LOC_ID --file "./achievement_badge.png"

# Release achievements to production
asc game-center achievements releases create --app APP_ID \
  --achievement-id ACH_ID
```

## Localization Management

Manage app metadata across locales.

```bash
# Download all localizations for a version
asc localizations download --version VERSION_ID --path "./localizations"

# Upload localizations from directory
asc localizations upload --version VERSION_ID --path "./localizations"

# List available localizations
asc localizations list --version VERSION_ID --paginate

# Set build-specific "What's New" for TestFlight
asc build-localizations create --build BUILD_ID --locale "de-DE" \
  --whats-new "Fehlerbehebungen und Verbesserungen"
```

## Migration from Fastlane

Migrate existing Fastlane metadata to asc format.

```bash
# Validate Fastlane metadata directory
asc migrate validate --fastlane-dir ./fastlane/metadata

# Import Fastlane metadata into App Store Connect
asc migrate import --app APP_ID --fastlane-dir ./fastlane/metadata

# Export current App Store Connect metadata to Fastlane format
asc migrate export --app APP_ID --output ./fastlane/metadata
```

## Offer Codes (Subscriptions)

Generate promotional offer codes for subscriptions.

```bash
# List offer code batches
asc offer-codes list --offer-code OFFER_ID --paginate

# Generate new codes
asc offer-codes generate --offer-code OFFER_ID \
  --quantity 100 --expiration-date "2026-06-30"

# Download generated codes
asc offer-codes values --id BATCH_ID --output "./promo-codes.txt"
```
