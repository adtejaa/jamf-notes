# Karabiner + Jamf Deployment Guide (Single Configuration Profile)

## Overview

This guide documents a working Jamf deployment for
**Karabiner-Elements** on macOS 13+ using a **single configuration
profile** containing:

-   PPPC (Input Monitoring -- ListenEvent)
-   System Extension (DriverKit)
-   Managed Login Items (Service Management)

**Team ID** G43BCU2T37

**Minimum Requirements** - macOS 13+ - Jamf Pro 10.42+ - Supervised
devices (ADE/ABM) for silent system extension approval

------------------------------------------------------------------------

# 1. Capture Code Requirement (Before Creating Profile)

Install Karabiner on a test Mac.

## Capture Core Service requirement:

``` bash
codesign -dr - "/Library/Application Support/org.pqrs/Karabiner-Elements/bin/karabiner_core_service"
```

## Capture EventViewer requirement (recommended):

``` bash
codesign -dr - "/Applications/Karabiner-EventViewer.app"
```

Copy the full requirement string exactly.

> Never manually construct the CodeRequirement.

------------------------------------------------------------------------

# 2. Create Configuration Profile in Jamf

Navigate to:

Jamf Pro → Computers → Configuration Profiles → New

**Profile Name** Karabiner -- Extensions + PPPC + LoginItems

**Payload Scope** System

This profile will contain three payloads.

------------------------------------------------------------------------

# 3. Payload 1 --- PPPC (Input Monitoring)

Add Payload: Privacy Preferences Policy Control

Service: ListenEvent

## Entry 1 --- Karabiner Core Service

  Setting            Value
  ------------------ ---------------------------------
  Identifier         org.pqrs.Karabiner-Core-Service
  Identifier Type    Bundle ID
  Authorization      Allow
  Code Requirement   (Paste captured requirement)

------------------------------------------------------------------------

## Entry 2 --- Karabiner EventViewer (Recommended)

  Setting            Value
  ------------------ --------------------------------
  Identifier         org.pqrs.Karabiner-EventViewer
  Identifier Type    Bundle ID
  Authorization      Allow
  Code Requirement   (Paste captured requirement)

------------------------------------------------------------------------

# 4. Payload 2 --- System Extension (DriverKit)

Add Payload: System Extensions

  Setting                Value
  ---------------------- -----------------------------------------------
  Team ID                G43BCU2T37
  Allowed Extension      org.pqrs.Karabiner-DriverKit-VirtualHIDDevice
  Allow User Overrides   Optional (disable for stricter control)

------------------------------------------------------------------------

# 5. Payload 3 --- Managed Login Items (Ventura+)

Add Payload: Login Items

Add Managed Login Item rule:

  Setting      Value
  ------------ ----------------
  Rule Type    TeamIdentifier
  Rule Value   G43BCU2T37

### Why TeamIdentifier?

-   Covers privileged + non-privileged daemons
-   Future-proof against label changes
-   Avoids fragile Label matching
-   Cleanest enterprise implementation

------------------------------------------------------------------------

# 6. Deployment Order (Critical)

1.  Deploy configuration profile
2.  Confirm profile is installed
3.  Install Karabiner package
4.  Reboot device

If Karabiner was previously installed:

``` bash
sudo sfltool resetbtm
sudo tccutil reset ListenEvent
```

Then reboot.

------------------------------------------------------------------------

# 7. Validation Checklist

## System Extension

``` bash
systemextensionsctl list
```

Expected: activated enabled

------------------------------------------------------------------------

## PPPC

``` bash
sudo sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db "SELECT client, service, auth_value FROM access WHERE client LIKE '%karabiner%';"
```

Expected: auth_value = 2

------------------------------------------------------------------------

## Managed Login Items

``` bash
sudo sfltool dumpbtm
```

Expected: Managed: YES

Or visually:

System Settings → General → Login Items

Karabiner entries should be: - Enabled - Greyed out - Not modifiable

------------------------------------------------------------------------

# 8. Common Issues

  -----------------------------------------------------------------------
  Symptom                                 Cause
  --------------------------------------- -------------------------------
  Extension "waiting for user"            Not supervised OR profile
                                          deployed after install

  auth_value = 0                          Incorrect CodeRequirement

  Privileged daemon asks admin password   Login Items rule mismatch

  Only one daemon managed                 Used Label instead of
                                          TeamIdentifier
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 9. Security Considerations

Karabiner requires:

-   Keystroke capture (Input Monitoring)
-   DriverKit virtual HID device
-   Background daemons

Recommended: - Scope to Engineering devices - Use Smart Groups for macOS
13+ - Restrict to required teams

------------------------------------------------------------------------

# 10. Minimal Required Identifiers

**Team ID** G43BCU2T37

**System Extension** org.pqrs.Karabiner-DriverKit-VirtualHIDDevice

**Core Service** org.pqrs.Karabiner-Core-Service

**EventViewer** org.pqrs.Karabiner-EventViewer

------------------------------------------------------------------------

# Final Architecture

One configuration profile containing:

-   com.apple.TCC.configuration-profile-policy
-   com.apple.system-extension-policy
-   com.apple.servicemanagement

Production-safe when:

-   Deployed before first app launch
-   Devices are supervised
-   CodeRequirements are captured correctly
