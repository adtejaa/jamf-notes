
# 🔐 Understanding CMS-Signed `.mobileconfig` Files on Apple Devices

## 📦 What is a `.mobileconfig` File?

A `.mobileconfig` file is a configuration profile used by Apple devices (macOS, iOS, iPadOS, tvOS) to apply settings automatically. These profiles are typically used to manage:

- Wi-Fi, VPN, or proxy settings  
- Device restrictions  
- Certificates (SCEP, PKCS12)  
- MDM enrollment  
- Custom enterprise configurations

They are written in **XML format** following the Apple **Property List (plist)** standard.

---

## 🔐 What is CMS Signing?

**CMS** stands for **Cryptographic Message Syntax** — a standard that defines how to digitally sign and optionally encrypt data.

When a `.mobileconfig` file is **CMS-signed**, it means:

- The original XML content is **wrapped inside a secure binary envelope**
- A **digital signature** is applied using a certificate (like a Developer ID or internal CA)
- The file is no longer human-readable — it looks like binary junk if opened in a text editor

CMS signing is used to **verify authenticity** and **protect against tampering**.

---

## ✅ Why CMS Signing is Used

| Purpose                        | Description |
|-------------------------------|-------------|
| **Integrity**                 | Confirms the profile wasn’t altered after signing |
| **Authenticity**             | Verifies the source (e.g., Apple, Jamf, your org) |
| **Security compliance**      | Required in high-security environments |
| **Required by Apple MDM**    | Especially for automated MDM enrollment and certificate payloads |

Signed profiles are especially important when:
- You're distributing MDM enrollment payloads
- You're installing profiles without user interaction
- You want to protect the payload (e.g., VPN credentials, certificates)

---

## 🔍 How to Identify a CMS-Signed `.mobileconfig`

1. **File type**:
   ```bash
   file profile.mobileconfig
   ```
   - If the output is `data`, it is CMS-signed (binary).
   - If it's `XML`, it's already unsigned.

2. **Visual check**: Open in a text editor — CMS-signed profiles look like gibberish/binary and **not readable XML**.

---

## 🛠 How to Decode (Remove Signature)

To view or edit a CMS-signed `.mobileconfig` file, you must **extract** the XML using:

```bash
security cms -D -i "signed.mobileconfig" > "unsigned.mobileconfig"
```

- `-D`: Decode
- `-i`: Input file
- Output: Plain-text XML profile

### 🔍 Validate the Output
```bash
plutil -lint unsigned.mobileconfig
```
- You should see: `unsigned.mobileconfig: OK`
- This confirms the file is valid XML and editable

---

## ⚖️ When to Use Signed vs. Unsigned Profiles

| Use Case                          | Requires Signing? | Notes |
|----------------------------------|--------------------|-------|
| Manual profile installation      | ❌ No              | Use unsigned |
| Custom config in Jamf            | ❌ No              | Jamf wraps it securely anyway |
| MDM enrollment                   | ✅ Yes             | macOS/iOS may reject unsigned MDM profiles |
| Delivering VPN or Certificates   | ✅ Recommended     | Protects sensitive credentials |
| Publicly distributed profiles    | ✅ Recommended     | Prevents tampering by end users |

---

## 🛑 Why You Can’t Just “Remove the Signature”

When CMS-signed:

- The file is no longer raw XML — it’s in a **binary CMS format**
- You can’t "manually remove the signature" by editing — there is no visible signature section to delete
- You must decode it using:
  ```bash
  security cms -D
  ```
- Any attempt to modify the binary directly will **corrupt the profile**

---

## ✍️ How to Sign a Profile (Advanced / Optional)

If you want to **sign a configuration profile**, use:

```bash
openssl smime -sign \
  -signer my_cert.pem \
  -inkey my_key.pem \
  -in unsigned.mobileconfig \
  -out signed.mobileconfig \
  -outform der -nodetach
```

You'll need:
- A valid certificate (`.pem`)
- Matching private key

This is useful if you're distributing profiles through email or outside of Jamf/MDM.

---

## ✅ Summary

- `.mobileconfig` files are Apple configuration profiles written in XML
- When CMS-signed, the XML is encrypted/wrapped in a binary envelope with a digital signature
- CMS-signed profiles are required for MDM enrollment and strongly recommended for distributing sensitive settings
- You must use `security cms -D` to decode CMS-signed profiles before editing
- After decoding, always validate using `plutil -lint`

---

## 📎 Helpful Terminal Commands

```bash
# Decode a signed profile to view/edit XML
security cms -D -i signed.mobileconfig > unsigned.mobileconfig

# Check if the decoded file is valid
plutil -lint unsigned.mobileconfig

# Check if a file is binary-signed
file signed.mobileconfig

# Install profile manually for testing
sudo profiles -I -F unsigned.mobileconfig

# Remove profile manually
sudo profiles -R -F unsigned.mobileconfig
```

---

## 🧪 Example Scenario

You downloaded a profile called `enrollmentProfile.mobileconfig` and it looks like binary when opened.

You decode it:
```bash
security cms -D -i enrollmentProfile.mobileconfig > decoded.mobileconfig
```

Now you can open `decoded.mobileconfig` in any text editor, make changes, and upload it to Jamf — or install it manually.

---
