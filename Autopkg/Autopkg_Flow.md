# AutoPkg + Jamf: Complete Summary

## 🧱 1. Creating an AutoPkg Recipe from Scratch
- A recipe includes metadata like `Identifier`, `Description`, `Input`, and a `Process` list.
- Processors execute in order and share variables using the `env` dictionary.

## 📦 2. Variables in Recipes (`%VAR%` Syntax)
- Variables like `%pathname%`, `%DOWNLOAD_URL%` are placeholders for values passed between processors.
- Syntax: `%VAR%` → gets resolved from the `Input` block or set during processing.

## 📥 3. `pathname`, `version`, and Sharing Across Recipes
- `pathname`: Set by `URLDownloader`, stores downloaded file path.
- `version`: Often set using `AppVersioner` or similar processors.
- All outputs can be reused in child recipes.

## 🔎 4. Viewing Set Variables
- View variables with `-v` flag or check receipt plist under AutoPkg cache.

## 🔗 4. `download_changed` Flag
- Controls whether the download is new:
  - `ETag`
  - `Last-Modified`
  - `Content-Length` (if `CHECK_FILESIZE_ONLY` is true)

## 🗃️ 8. Where Headers Are Stored
- Metadata stored in:
  - `.info.json` file (`URLDownloaderPython`)
  - Extended attributes (`URLDownloader`)
  - `receipts.plist`

## 🔁 9. Header Handling
- AutoPkg sets xattrs like:
  - `com.github.autopkg.etag`
  - `com.github.autopkg.last-modified`
- Used in conditional HTTP GET headers:
  - `If-None-Match`
  - `If-Modified-Since`

## 📜 10. How the GET Request Works
1. Sends conditional headers.
2. If 304 Not Modified → skips.
3. If modified → downloads new version.

## 🧩 11. Source Code Confirmation
- URLDownloader uses `xattr.setxattr()` and `store_headers()` to persist headers.

## 🧠 12. Extended Attributes (xattr)
| Attribute | Set By | Purpose |
|----------|--------|---------|
| `com.github.autopkg.etag` | AutoPkg | Stores ETag |
| `com.github.autopkg.last-modified` | AutoPkg | Last-Modified |
| `com.apple.diskimages.recentcksum` | macOS | System-level DMG checksum |

---

## ✅ `URLDownloader` vs `URLDownloaderPython`

| Feature | URLDownloader | URLDownloaderPython |
|--------|---------------|---------------------|
| Language | Python + macOS | Pure Python |
| Metadata Storage | xattr | .info.json |
| Header Checking | ✅ | ✅ |
| File Reuse | Based on headers | Based on JSON |
| Cross-platform | No | Yes |

---

## 🔹 Key Optional Inputs

### 1. `CHECK_FILESIZE_ONLY`
- Default: `False`
- If `True`, only compares `Content-Length`
- Use for mirrors/CDNs with fluctuating ETag headers

### 2. `DOWNLOAD_MISSING_FILE`
- Default: `True`
- If file is missing but metadata exists:
  - `True` → re-download
  - `False` → skip silently

### 3. `BYPASS_STOP_PROCESSING_IF_DOWNLOAD_UNCHANGED`
- Default: `False`
- Used by `StopProcessingIfDownloadUnchanged`
- If `True`, continues processing even if file hasn’t changed

---

## ✅ Jamf Recipe Execution Sequence

1. **Parent Recipe (.download or .pkg)**
    - Downloads app
    - Verifies and builds package
    - Sets `pathname`, `version`, etc.

2. **Child Recipe (.jamf)**
    - Uses:
        - `JamfUploaderPackage`: Uploads package
        - `JamfUploaderIcon`: Uploads icon
        - `JamfUploaderGroup`: Smart group
        - `JamfUploaderPolicy`: Creates Self Service policy
    - Inputs used:
        - `SOFTWARE_TITLE`, `CATEGORY`, `POLICY_NAME`, etc.

3. **Execution Order Matters**
    - Package must upload before policy creation
    - Each processor builds on previous ones



# Jamf AutoPkg Recipe Processor Breakdown

This document summarizes the sequence and purpose of processors in a Jamf recipe and explains conditional logic using `StopProcessingIf`.

---

## 🧰 Recipe Overview

**Identifier**: `com.github.smithjw-actions.jamf.Talon`  
**Parent Recipe**: `com.github.smithjw-actions.pkg.Talon`  
**Minimum Version**: 2.3

---

## 🧩 Input Variables

These values define how the package and policy are handled in Jamf:

```yaml
NAME: Talon
SOFTWARE_TITLE: '%NAME%'
CATEGORY: Apps
POLICY_CATEGORY: Apps
POLICY_CUSTOM_TRIGGER: install-%SOFTWARE_TITLE%
POLICY_NAME: '%NAME%'
POLICY_RUN_COMMAND: ' '
POLICY_RUN_RECON: 'false'
POLICY_SCOPE_ALL_COMPUTERS: 'true'
POLICY_TEMPLATE: Policy_Template-Self_Service.xml
REMOVE_OLD_PACKAGES: 'true'
SELF_SERVICE_AVAILABLE: 'true'
SELF_SERVICE_DESCRIPTION: ''
SELF_SERVICE_DISPLAY_NAME: '%NAME%'
SELF_SERVICE_ICON: '%SOFTWARE_TITLE%.png'
SELF_SERVICE_INSTALL_BUTTON: Install
SELF_SERVICE_REINSTALL_BUTTON: Install
REPLACE_POLICY: 'true'
PACKAGE_INFO: autopkg
```

---

## ⚙️ Processor Sequence and Purpose

### 1. `JamfPackageUploader`
- **Processor**: `com.github.grahampugh.jamf-upload.processors/JamfPackageUploader`
- **Purpose**: Uploads the generated `.pkg` to Jamf.
- **Inputs**:
  - `pkg_category`: `%CATEGORY%`
  - `pkg_info`: `%PACKAGE_INFO%`

---

### 2. `JamfPolicyUploader`
- **Processor**: `com.github.grahampugh.jamf-upload.processors/JamfPolicyUploader`
- **Purpose**: Creates/updates a policy in Jamf using a template.
- **Inputs**:
  - `icon`: `%SELF_SERVICE_ICON%`
  - `policy_name`: `%POLICY_NAME%`
  - `policy_template`: `%POLICY_TEMPLATE%`
  - `replace_policy`: `%REPLACE_POLICY%`

---

### 3. `LastRecipeRunResult`
- **Processor**: `com.github.grahampugh.recipes.postprocessors/LastRecipeRunResult`
- **Purpose**: Outputs a summary of what this recipe did (for logging/reporting).

---

### 4. `StopProcessingIf`
- **Processor**: `StopProcessingIf`
- **Purpose**: Acts as a conditional gate to control cleanup.
- **Input**:
  - `predicate`: `%REMOVE_OLD_PACKAGES% == false`
- **Effect**:  
  If `REMOVE_OLD_PACKAGES` is false, this stops the recipe here.  
  Else, continues to the cleanup step.

---

### 5. `JamfPackageCleaner`
- **Processor**: `com.github.grahampugh.jamf-upload.processors/JamfPackageCleaner`
- **Purpose**: Deletes old versions of the uploaded package from Jamf.
- **Inputs**:
  - `pkg_name_match`: `%SOFTWARE_TITLE%-`
  - `versions_to_keep`: `2`

---
