# Play Store Publishing Workflows

End-to-end workflow examples for common Google Play publishing scenarios.

## Simple Release to Production

Upload an AAB directly to production (for apps with established track record).

```bash
# 1. Validate first (dry run)
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --validate_only \
  --json_key service-account.json \
  --package_name com.example.app

# 2. Upload for real
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --json_key service-account.json \
  --package_name com.example.app
```

## Full Staged Rollout (Recommended for Production)

Gradually roll out to production with monitoring at each stage.

```bash
# Step 1: Upload to internal track for QA
fastlane supply \
  --aab path/to/app.aab \
  --track internal \
  --json_key service-account.json \
  --package_name com.example.app

# Step 2: After QA validation, promote to beta
fastlane supply \
  --track internal \
  --track_promote_to beta \
  --json_key service-account.json \
  --package_name com.example.app

# Step 3: Monitor beta for 24-48 hours, then promote to production at 10%
fastlane supply \
  --track beta \
  --track_promote_to production \
  --track_promote_release_status inProgress \
  --rollout 0.1 \
  --json_key service-account.json \
  --package_name com.example.app

# Step 4: Increase to 25% if metrics look good
fastlane supply \
  --track production \
  --rollout 0.25 \
  --release_status inProgress \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app

# Step 5: Increase to 50%
fastlane supply \
  --track production \
  --rollout 0.5 \
  --release_status inProgress \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app

# Step 6: Full rollout
fastlane supply \
  --track production \
  --rollout 1.0 \
  --release_status completed \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app
```

## Emergency Halt

Stop a problematic release immediately.

```bash
fastlane supply \
  --track production \
  --release_status halted \
  --skip_upload_aab \
  --skip_upload_metadata \
  --json_key service-account.json \
  --package_name com.example.app
```

**After halting:**
- Fix the issue, build a new AAB
- Upload as a new release to internal track
- Go through the staged rollout flow again

## Metadata-Only Update

Update store listing without uploading a new binary.

```bash
# 1. Initialize metadata directory (first time only)
fastlane supply init \
  --json_key service-account.json \
  --package_name com.example.app

# 2. Edit text files locally:
#    fastlane/metadata/android/en-US/title.txt
#    fastlane/metadata/android/en-US/short_description.txt
#    fastlane/metadata/android/en-US/full_description.txt

# 3. Upload metadata only
fastlane supply \
  --skip_upload_aab \
  --skip_upload_apk \
  --json_key service-account.json \
  --package_name com.example.app
```

## Multi-Locale Metadata Update

Update store listings for multiple languages.

```bash
# 1. Initialize to download existing metadata
fastlane supply init \
  --json_key service-account.json \
  --package_name com.example.app

# 2. Edit metadata for each locale:
#    fastlane/metadata/android/en-US/full_description.txt
#    fastlane/metadata/android/de-DE/full_description.txt
#    fastlane/metadata/android/ja-JP/full_description.txt
#    etc.

# 3. Upload all metadata (images will sync via SHA256 hash comparison)
fastlane supply \
  --skip_upload_aab \
  --skip_upload_apk \
  --sync_image_upload \
  --json_key service-account.json \
  --package_name com.example.app
```

## Screenshot Update

Update screenshots without uploading a new build.

```bash
# 1. Place screenshots in correct directories:
#    fastlane/metadata/android/en-US/images/phoneScreenshots/01_home.png
#    fastlane/metadata/android/en-US/images/phoneScreenshots/02_feature.png
#    (alphabetical order = display order)

# 2. Upload screenshots only
fastlane supply \
  --skip_upload_aab \
  --skip_upload_apk \
  --skip_upload_metadata \
  --skip_upload_changelogs \
  --json_key service-account.json \
  --package_name com.example.app
```

**Note:** If using >2 skip flags via CLI, some may be silently ignored. Use a Fastfile instead:

```ruby
# fastlane/Fastfile
lane :update_screenshots do
  upload_to_play_store(
    skip_upload_aab: true,
    skip_upload_apk: true,
    skip_upload_metadata: true,
    skip_upload_changelogs: true,
    json_key: "service-account.json",
    package_name: "com.example.app"
  )
end
```

## Upload with ProGuard/R8 Mapping

Include deobfuscation mapping for crash reporting.

```bash
fastlane supply \
  --aab path/to/app.aab \
  --mapping path/to/mapping.txt \
  --track internal \
  --json_key service-account.json \
  --package_name com.example.app
```

For multiple APKs with separate mappings:

```bash
fastlane supply \
  --apk_paths "app-arm64.apk,app-armeabi.apk" \
  --mapping_paths "mapping-arm64.txt,mapping-armeabi.txt" \
  --track internal \
  --json_key service-account.json \
  --package_name com.example.app
```

## In-App Update Priority

Set update urgency for Android's in-app update API.

```bash
# Priority 5 = highest urgency (forces immediate update prompt)
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --in_app_update_priority 5 \
  --json_key service-account.json \
  --package_name com.example.app
```

| Priority | Behavior |
|----------|----------|
| 0 (default) | No special update prompt |
| 1-2 | Low priority, flexible update suggested |
| 3 | Medium priority |
| 4-5 | High priority, immediate update strongly encouraged |

## Draft Release

Create a draft release that can be reviewed in Play Console before going live.

```bash
fastlane supply \
  --aab path/to/app.aab \
  --track production \
  --release_status draft \
  --json_key service-account.json \
  --package_name com.example.app
```

**Note:** Draft apps (never published) can only create `draft` releases until all Play Console setup questionnaires are complete.

## CI/CD Pipeline Example

Typical flow for automated deployment:

```bash
# 1. Build the app (example with Flutter)
flutter build appbundle --release

# 2. Validate upload
fastlane supply \
  --aab build/app/outputs/bundle/release/app-release.aab \
  --track internal \
  --validate_only \
  --json_key "$SUPPLY_JSON_KEY" \
  --package_name com.example.app

# 3. Upload to internal track
fastlane supply \
  --aab build/app/outputs/bundle/release/app-release.aab \
  --track internal \
  --json_key "$SUPPLY_JSON_KEY" \
  --package_name com.example.app

# 4. (After manual QA) Promote to production via separate pipeline trigger
fastlane supply \
  --track internal \
  --track_promote_to production \
  --track_promote_release_status inProgress \
  --rollout 0.1 \
  --json_key "$SUPPLY_JSON_KEY" \
  --package_name com.example.app
```

## Using json_key_data (CI Without Files)

Pass JSON key content directly (useful when secrets are env vars, not files):

```bash
# Store JSON content in env var (e.g., from CI secret)
export SUPPLY_JSON_KEY_DATA='{"type":"service_account","project_id":"..."}'

fastlane supply \
  --aab path/to/app.aab \
  --track internal \
  --package_name com.example.app
# Picks up SUPPLY_JSON_KEY_DATA automatically
```

## Validate Service Account Credentials

Test that your service account is properly configured:

```bash
# Using fastlane's built-in validation
fastlane run validate_play_store_json_key json_key:service-account.json

# Or try an init (downloads metadata if credentials work)
fastlane supply init \
  --json_key service-account.json \
  --package_name com.example.app
```
