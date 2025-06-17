# Windows Family Safety Bypass via Race Condition

## Overview

This write-up describes a local bypass of Microsoft Family Safety restrictions on Windows, discovered through timing and process race conditions during session initialization.

**Reported to:** Microsoft Security Response Center (MSRC)  
**MSRC Response:** Acknowledged, assessed as low severity (no CVE issued)  
**Disclosure Status:** Responsible, no NDA  
**Status:** Not serviced, shared with product team  

---

## Summary

A restricted (child) user account on Windows can bypass Family Safety restrictions by exploiting a race condition that occurs during user login, where enforcement services have not fully initialized.

---

## Reproduction Steps

1. Log into a child/restricted user account
2. While Family Safety services are still loading:
    - Rapidly launch previously restricted apps
3. Immediately disconnect from the internet
4. *(Optional but effective)* Suspend or overload `WpcMon.exe` (without requiring admin access)
5. Observe that:
    - `ms-wpc://apptime?...` popup appears
    - All app restrictions fail open
    - Access to previously blocked applications is allowed
6. Restrictions reapply only after popup dismissal or system reboot

---

## Technical Explanation

The issue appears to stem from a race condition in the Family Safety enforcement services (`WpcMon.dll`, `WpcSvc.dll`) that are not fully initialized at login time.

### Key Components Involved

- **`WpcMon.dll`** – Monitors application launches
- **`WpcSvc.dll`** – Family Safety service logic
- **`ms-wpc://apptime?...`** – Internal URI triggered on enforcement failures

### Hypothetical Call Stack / Flow

```plaintext
[explorer.exe] or [app.exe]
  └──> COM: AppContainer Policy Manager
        └──> WpcMon.dll
              └──> WpcSvc.dll
                    └──> Cloud sync or policy check fails
                          └──> Internal URI triggered: ms-wpc://apptime
                                └──> Notification appears
                                      └──> Enforcement state drops (FAIL OPEN)
