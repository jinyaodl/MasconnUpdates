---
description: Automate GitHub app release, APK upload, and apps.json metadata update
---

This workflow automates the process of creating a GitHub release for an application (APK) and updating the local `apps.json` metadata.

// turbo-all

### Metadata Extraction
This workflow assumes the filename follows the convention: `[APP_NAME]_[VERSION_CODE].apk` (e.g., `MasconnLauncher_101.apk`).

1. **Parse Filename**
   - Extract `APP_NAME`: Everything before the underscore.
   - Extract `VERSION_CODE`: Everything between the underscore and `.apk`.
   - Set `VERSION_TAG`: Use `app/[APP_NAME]/[VERSION_CODE]` (e.g., `app/MasconnLauncher/101`).

2. **Calculate SHA256 Checksum**
   Run `shasum -a 256 dist/[FILENAME]` to get the hash.

3. **Create GitHub Release**
   Run `gh release create [VERSION_TAG] -R jinyaodl/iwantacake --title "[APP_NAME] [VERSION_CODE]" --notes "Automated release for [APP_NAME]"` in the repository.

4. **Upload App Asset**
   Run `gh release upload [VERSION_TAG] "dist/[FILENAME]" -R jinyaodl/iwantacake` to attach the file to the release.

5. **Retrieve Download URL**
   Run `gh release view [VERSION_TAG] -R jinyaodl/iwantacake --json assets --jq '.assets[] | select(.name=="[FILENAME]") | .browser_download_url'` to get the public URL.

6. **Update apps.json**
   Locate the entry for `package_name: [PACKAGE_NAME]` or `app_name: [APP_NAME]` and update:
   - `download_url`: Use URL from step 5.
   - `sha256_checksum`: Use hash from step 2.
   - `version_code`: Use extracted version from step 1.
   - `version_name`: Set based on the new version.

7. **Verify and Commit**
   - Run `jq . apps.json` to verify JSON syntax.
   - Run `git add apps.json && git commit -m "chore: release app [VERSION_TAG]"`

8. **Cleanup**
   Run `rm dist/[FILENAME]` to remove the local binary.
