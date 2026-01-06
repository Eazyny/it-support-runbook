# Windows Mapped Drive Login Issue After Password Change

## Symptom
Mapped drive shows unavailable and prompts for credentials. New password doesn’t work.

## Likely Cause
Windows cached old credentials / session token.

## Fix
1. Sign into Windows on the main PC using the **password** (not PIN) at least once
2. Retry access to the share
3. If still failing:
   - Control Panel → Credential Manager → remove old saved credentials for the server
   - Re-map the drive

## Verify
Drive opens normally and reconnects after reboot.
