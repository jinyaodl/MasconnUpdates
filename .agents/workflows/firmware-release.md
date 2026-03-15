---
description: Automate GitHub firmware release, file upload, and JSON metadata update
---

This workflow automates the process of creating a GitHub release for a firmware binary and updating the local metadata file.


// turbo-all


### Metadata Extraction
This workflow assumes the filename follows the convention: `${product_id}_${version_code}.zip` (e.g., `XT-X1-PRO_202.zip`).

1. **Parse Filename**
   - Extract `PRODUCT_ID`: Everything before the underscore.
   - Extract `VERSION_CODE`: Everything between the underscore and `.zip`.
   - Set `VERSION_TAG`: Use the full filename (without extension) as the tag.

2. **Calculate SHA256 Checksum (Remote)**
   Run `gh release download [VERSION_TAG] -p "[FILENAME]" -O - -R jinyaodl/iwantacake | shasum -a 256` to get the hash directly from the remote asset.

3. **Create GitHub Release**
   Run `gh release create [VERSION_TAG] -R jinyaodl/iwantacake --title "Release [VERSION_TAG]" --notes "Automated release for [PRODUCT_ID]"` in the repository.

4. **Upload Firmware Asset**
   Run `gh release upload [VERSION_TAG] "dist/[FILENAME]" -R jinyaodl/iwantacake` to attach the file to the release.

5. **Retrieve Download URL**
   Run `gh release view [VERSION_TAG] -R jinyaodl/iwantacake --json assets --jq '.assets[] | select(.name=="[FILENAME]") | .browser_download_url'` to get the public URL.

6. **Update firmware_update.json**
   Locate the entry for `product_id: [PRODUCT_ID]` and update:
   - `download_url`: Use URL from step 5.
   - `sha256_checksum`: Use hash from step 2.
   - `version_code`: Use extracted version from step 1.
   - `version_name`: Set to `[VERSION_TAG]`.

7. **Verify and Commit**
   - Run `jq . firmware_update.json` to verify JSON syntax.
   - Run `git add firmware_update.json && git commit -m "chore: release [VERSION_TAG] for [PRODUCT_ID]"`

8. **Cleanup**
   Run `rm dist/[FILENAME]` to remove the local binary.
