# Managed Background Tasks (macOS 13+)

## Introduction

Starting with **macOS 13 (Ventura)**, Apple redesigned how applications
register and manage background services. Instead of relying on
traditional launch agents and launch daemons placed directly into system
directories, Apple introduced the **SMAppService framework**.

This modern framework allows applications to declare helper executables
within their own app bundles, improving:

-   System integrity
-   Transparency
-   User visibility
-   Administrative control

These helper components can include:

-   Login items (applications that start at user login)
-   Launch agents (user-level background services)
-   Launch daemons (system-level background services)
-   Privileged background processes
-   Menu bar utilities and auxiliary services

All of these now surface in a centralized location:

System Settings → General → Login Items

Within this interface, macOS displays:

-   **Open at Login** applications
-   **Allow in the Background** services

By default, users (and administrators) can toggle many of these services
on or off. However, disabling certain background components can cause
applications to malfunction, lose functionality, or fail entirely.

To address this, Apple introduced **MDM-based enforcement** through the
`com.apple.servicemanagement` payload.

In Jamf Pro, this is exposed as the **Managed Login Items** payload ---
but technically it governs *all* SMAppService background tasks, not just
login items.

------------------------------------------------------------------------

## Why Managed Background Tasks Matter

Managed Background Tasks allow organizations to:

-   Enforce required background services
-   Prevent users from disabling critical helpers
-   Remove administrative credential prompts when toggling privileged
    daemons
-   Ensure consistent application behavior across managed fleets
-   Maintain compliance with security and operational standards

From an enterprise perspective, Managed Background Tasks are now a core
component of modern macOS application deployment.

They complement:

-   **PPPC (Privacy Preferences Policy Control)** for permission
    management
-   **System Extensions payloads** for driver approvals
-   Traditional configuration profiles for application configuration

When properly configured, the Service Management payload ensures
required application helpers remain:

-   Active
-   Enforced
-   Protected
-   Non-modifiable

--- without degrading the user experience.

------------------------------------------------------------------------

# Service Management Payload in Jamf Pro

------------------------------------------------------------------------

## 1. Background Tasks in macOS 13+

### What Changed

Before macOS 13, helper processes were installed into:

-   `/Library/LaunchAgents`
-   `/Library/LaunchDaemons`
-   `~/Library/LaunchAgents`

Starting macOS 13, Apple introduced **SMAppService**, which:

-   Allows apps to register helpers inside the app bundle
-   Eliminates the need to write to protected system directories
-   Centralizes background task visibility and control

These background components now appear in:

System Settings → General → Login Items

------------------------------------------------------------------------

## 2. What Are Managed Background Tasks?

Background tasks include:

-   LaunchAgents
-   LaunchDaemons
-   Login Items
-   App-registered helpers via SMAppService

### Without MDM Management

-   Users can toggle them off
-   Privileged daemons require admin credentials
-   Applications may break if disabled

### With MDM (Service Management Payload)

-   Items are marked **Managed**
-   Toggles are greyed out
-   Users cannot disable them
-   No admin authentication prompt appears

------------------------------------------------------------------------

## 3. Jamf Service Management Payload

Jamf Pro 10.42+ supports the Apple payload:

`com.apple.servicemanagement`

This payload allows administrators to define **Rules** that match
background items and enforce them.

------------------------------------------------------------------------

## 4. Available Rule Types

  Rule Type                What It Matches
  ------------------------ ----------------------
  BundleIdentifier         Exact app bundle ID
  BundleIdentifierPrefix   Bundle ID prefix
  Label                    Exact launchd label
  LabelPrefix              Launchd label prefix
  TeamIdentifier           Code signing Team ID

Each rule determines which background services are enforced.

------------------------------------------------------------------------

## 5. Identifying Background Tasks

To inspect currently registered background items:

``` bash
sudo sfltool dumpbtm
```

This command outputs:

-   Label
-   BundleIdentifier
-   TeamIdentifier
-   Managed status

------------------------------------------------------------------------

## 6. Choosing the Correct Rule Type

### Label

Use when: - Managing a specific launch daemon - Precision is required

### BundleIdentifier

Use when: - Managing login items tied to a specific application

### TeamIdentifier

Use when: - An app installs multiple background helpers - You want
future-proof enforcement - You want to manage all services signed by a
vendor

TeamIdentifier is often the cleanest enterprise implementation when
managing complex applications.

------------------------------------------------------------------------

## 7. Creating Managed Background Tasks in Jamf

### Step 1 --- Create Configuration Profile

Jamf Pro → Computers → Configuration Profiles → New

### Step 2 --- Add Login Items Payload

Select:

Login Items

(This corresponds to `com.apple.servicemanagement`.)

### Step 3 --- Add Rule

Click:

Add Managed Login Item

Configure:

  Field        Value
  ------------ --------------------
  Rule Type    TeamIdentifier
  Rule Value   `<Vendor Team ID>`

Save the profile and scope appropriately.

------------------------------------------------------------------------

## 8. Verifying Managed Background Tasks

### GUI Validation

System Settings → General → Login Items

Managed items will:

-   Be enabled
-   Have greyed-out toggles
-   Not allow modification

### Terminal Validation

``` bash
sudo sfltool dumpbtm
```

Look for:

Managed: YES

### Reset for Testing

If the profile was applied after installation:

``` bash
sudo sfltool resetbtm
```

Reboot recommended.

------------------------------------------------------------------------

## 9. Example: Karabiner

Karabiner installs:

-   Non-Privileged Agent
-   Privileged Daemon

To manage all its background services:

RuleType: TeamIdentifier\
RuleValue: G43BCU2T37

------------------------------------------------------------------------

## 10. Relationship to Other Payloads

Managed Background Tasks do **not** replace:

-   PPPC (Privacy Preferences Policy Control)
-   System Extensions

For apps like Karabiner:

  Payload              Purpose
  -------------------- -----------------------------
  PPPC                 Allow keystroke capture
  System Extension     Approve driver
  Service Management   Enforce background services

All are required for full functionality.
