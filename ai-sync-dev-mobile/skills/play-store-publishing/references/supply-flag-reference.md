# Fastlane Supply - Complete Flag & Option Reference

All available flags and options for `fastlane supply` (aka `upload_to_play_store`).

## Authentication

| Flag | Env Var | Description |
|------|---------|-------------|
| `--json_key PATH` | `SUPPLY_JSON_KEY` | Path to service account JSON key file |
| `--json_key_data DATA` | `SUPPLY_JSON_KEY_DATA` | Raw JSON key data (for CI, avoids file) |
| `--root_url URL` | `SUPPLY_ROOT_URL` | Override Google API URL (default: `https://www.googleapis.com/`) |
| `--timeout SECONDS` | `SUPPLY_TIMEOUT` | Timeout for HTTP requests |

## Binary Upload

| Flag | Env Var | Description |
|------|---------|-------------|
| `--apk PATH` | `SUPPLY_APK` | Path to APK file |
| `--apk_paths PATH,PATH` | `SUPPLY_APK_PATHS` | Multiple APK paths (comma-separated) |
| `--aab PATH` | `SUPPLY_AAB` | Path to AAB (Android App Bundle) |
| `--aab_paths PATH,PATH` | `SUPPLY_AAB_PATHS` | Multiple AAB paths |
| `--mapping PATH` | `SUPPLY_MAPPING` | Path to ProGuard mapping file |
| `--mapping_paths PATH,PATH` | `SUPPLY_MAPPING_PATHS` | Multiple mapping file paths |

## Track & Release

| Flag | Env Var | Description |
|------|---------|-------------|
| `--track NAME` | `SUPPLY_TRACK` | Target track: `internal`, `alpha`, `beta`, `production` |
| `--track_promote_to NAME` | `SUPPLY_TRACK_PROMOTE_TO` | Promote current track's version to this track |
| `--track_promote_release_status STATUS` | `SUPPLY_TRACK_PROMOTE_RELEASE_STATUS` | Status after promotion |
| `--rollout FLOAT` | `SUPPLY_ROLLOUT` | Rollout fraction: `0.0` to `1.0` |
| `--release_status STATUS` | `SUPPLY_RELEASE_STATUS` | Release status (see below) |
| `--version_name NAME` | `SUPPLY_VERSION_NAME` | Custom version name |
| `--version_code CODE` | `SUPPLY_VERSION_CODE` | Specific version code to operate on |
| `--version_codes_to_retain CODES` | `SUPPLY_VERSION_CODES_TO_RETAIN` | Comma-separated version codes to keep active |

### Release Status Values

| Status | Meaning |
|--------|---------|
| `draft` | Not visible to users, editable |
| `completed` | Full rollout to all users (default since Fastlane 2.226.0+) |
| `inProgress` | Staged rollout at specified percentage |
| `halted` | Paused rollout, no new users get update |

## Package Identification

| Flag | Env Var | Description |
|------|---------|-------------|
| `--package_name NAME` | `SUPPLY_PACKAGE_NAME` | Android package name (e.g., `com.example.app`) |

## Metadata

| Flag | Env Var | Description |
|------|---------|-------------|
| `--metadata_path PATH` | `SUPPLY_METADATA_PATH` | Path to metadata directory (default: `fastlane/metadata/android`) |
| `--changes_not_sent_for_review BOOL` | `SUPPLY_CHANGES_NOT_SENT_FOR_REVIEW` | If true, changes saved as draft |

## Skip Flags

| Flag | Env Var | Description |
|------|---------|-------------|
| `--skip_upload_aab` | `SUPPLY_SKIP_UPLOAD_AAB` | Skip AAB upload |
| `--skip_upload_apk` | `SUPPLY_SKIP_UPLOAD_APK` | Skip APK upload |
| `--skip_upload_metadata` | `SUPPLY_SKIP_UPLOAD_METADATA` | Skip metadata (title, description, etc.) |
| `--skip_upload_images` | `SUPPLY_SKIP_UPLOAD_IMAGES` | Skip all images |
| `--skip_upload_screenshots` | `SUPPLY_SKIP_UPLOAD_SCREENSHOTS` | Skip screenshots only |
| `--skip_upload_changelogs` | `SUPPLY_SKIP_UPLOAD_CHANGELOGS` | Skip changelog files |

**Known issue:** Using more than 2 `--skip*` flags on CLI may be silently ignored due to a parsing bug. Use a Fastfile for complex skip combinations.

## Advanced Options

| Flag | Env Var | Description |
|------|---------|-------------|
| `--validate_only` | `SUPPLY_VALIDATE_ONLY` | Validate without publishing changes |
| `--in_app_update_priority N` | `SUPPLY_IN_APP_UPDATE_PRIORITY` | Priority 0-5 for Android in-app update API |
| `--ack_bundle_installation_warning` | `SUPPLY_ACK_BUNDLE_INSTALLATION_WARNING` | Acknowledge large bundle warning (>150MB) |
| `--sync_image_upload` | `SUPPLY_SYNC_IMAGE_UPLOAD` | Compare SHA256 hashes, upload only changed images |

## OBB (Expansion Files)

| Flag | Description |
|------|-------------|
| `--obb_main_references_version CODE` | Reference existing main OBB from version code |
| `--obb_main_file_size BYTES` | Size of referenced main OBB |
| `--obb_patch_references_version CODE` | Reference existing patch OBB from version code |
| `--obb_patch_file_size BYTES` | Size of referenced patch OBB |

**Note:** OBB files placed alongside your APK with `main` or `patch` in the filename are uploaded automatically.

## Environment Variables

All flags can be set via environment variables with the `SUPPLY_` prefix. Additionally:

| Variable | Purpose |
|----------|---------|
| `FL_NUMBER_OF_THREADS` | Concurrent metadata upload threads (1-10) |
| `DELIVER_NUMBER_OF_THREADS` | Same as above (alias) |

### Using .env Files

Fastlane loads `.env`, `.env.default`, and `.env.<environment>` from the Fastfile directory:

```bash
# Run with specific environment
fastlane supply --env production

# Loads: .env, .env.default, .env.production
```

## Metadata Directory Structure

```
fastlane/metadata/android/
├── en-US/
│   ├── title.txt              # App title (30 chars max on Play Store)
│   ├── short_description.txt  # 80 chars max
│   ├── full_description.txt   # 4000 chars max
│   ├── video.txt              # YouTube promo video URL
│   ├── changelogs/
│   │   ├── default.txt        # Default changelog for all versions
│   │   └── 123.txt            # Version code-specific changelog
│   └── images/
│       ├── icon.png           # 512x512 app icon
│       ├── featureGraphic.png # 1024x500 feature graphic
│       ├── phoneScreenshots/  # Phone screenshots (2-8 images)
│       ├── sevenInchScreenshots/
│       ├── tenInchScreenshots/
│       ├── tvScreenshots/
│       └── wearScreenshots/
├── de-DE/
│   └── ... (same structure)
└── ja-JP/
    └── ... (same structure)
```

### Screenshot Ordering

Screenshots are uploaded in alphabetical order by filename. Use numeric prefixes:

```
phoneScreenshots/
├── 01_welcome.png
├── 02_dashboard.png
├── 03_settings.png
```

### Changelog Version Code Matching

The changelog filename must exactly match the version code of the APK/AAB:

```
changelogs/
├── default.txt    # Used when no version-specific file exists
├── 42.txt         # Used for version code 42
└── 43.txt         # Used for version code 43
```
