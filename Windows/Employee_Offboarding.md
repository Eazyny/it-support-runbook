## Goal
Protect company data, prevent unauthorized access, and cleanly transition ownership of accounts/files/devices.

## Trigger
HR/Manager request (termination or resignation) with:
- Full name + username/email
- Last working day + exact time (important)
- Manager + who receives access
- Any special systems (finance/admin/VIP)

---

## 0) Before you touch anything: decide the urgency
### Same-day / immediate termination
Do security actions **immediately** (Steps 1–4 first). Don’t wait.

### Scheduled resignation
Prep ahead of time, then execute at the agreed cutoff time.

---

## 1) Secure the identity + collect context
Open a ticket checklist with:
- User email / username
- Manager email
- Department
- Devices assigned (asset tags)
- Key apps access (VPN, ticketing, Slack/Teams, CRM, etc.)
- Whether mailbox/OneDrive must be preserved and for how long

---

## 2) Disable sign-in
### Active Directory (ADUC)
Open: **Server Manager → Tools → Active Directory Users and Computers**

1. Find user
2. Right-click → **Disable Account**
3. Open **Properties → Account**
   - Confirm it shows disabled

**Notes**
- Disabling is safer than deleting. Deleting breaks auditing and access to files/mailbox references.

---

## 3) Reset password (optional but recommended for high-risk exits)
In **ADUC**:
1. Right-click user → **Reset Password**
2. Set a strong random password
3. OK

This helps kill any remembered credentials quickly.

---

## 4) Revoke active sessions + MFA (M365 / Entra Admin Center)
Open: **Microsoft 365 Admin Center** and/or **Entra ID (Azure AD) Admin Center**

Do these (wording varies slightly by portal):
- **Block sign-in** for the user
- **Sign out of all sessions** / revoke refresh tokens
- **Reset MFA methods** (or remove authenticator devices)
- If using Conditional Access: confirm user is blocked

**Why:** Disabling AD alone may not stop cloud sessions instantly.

---

## 5) Transfer ownership: mailbox + OneDrive
### Mailbox access (Exchange Admin Center / M365 Admin)
Depending on your org:
- Convert mailbox to **Shared Mailbox** (common approach)
- Give manager delegated access (Full Access) if approved
- Set forwarding (only if policy allows)

### OneDrive (SharePoint Admin Center)
- Create/confirm a **OneDrive access link for the manager**
- Set retention/ownership policy as required

**Best practice:** don’t forward everything forever. Use shared mailbox + retention.

---

## 6) Remove from groups (clean access control)
Back in **ADUC → user Properties → Member Of**
- Screenshot or note groups first (for audit)
- Remove from sensitive groups:
  - Admin / privileged groups
  - Finance/HR groups
  - VPN/Remote access groups
  - Any app-based access groups

**Tip:** If you’re disabling the account immediately, group removal can be done after—but still do it.

---

## 7) Disable/Remove access in key systems (GUI-based checklist)
This depends on your environment, but a good baseline list:

- **VPN portal** (remove user / disable account)
- **Ticketing system** (set to inactive, reassign open tickets)
- **Slack / Teams** (disable/deactivate, transfer ownership of channels if needed)
- **GitHub / GitLab** (remove from org, rotate keys/tokens if needed)
- **Password manager** (remove user, rotate shared vault secrets)
- **CRM / Finance tools** (deactivate account, transfer records)

**High-risk rule:** if they had admin access anywhere, rotate shared credentials.

---

## 8) Device + access recovery
### If you manage devices (Intune / endpoint portal)
- Mark the device as **lost** (if needed)
- **Wipe** if termination is immediate and device is not being returned promptly
- Confirm BitLocker/device encryption status (org dependent)

### Physical recovery (office)
- Collect:
  - Laptop + charger
  - Badge
  - Keys
  - Headset/dock (optional)
- Confirm device is returned to IT inventory

---

## 9) Shared resources handoff
Ask the manager:
- Who becomes owner of:
  - Shared drives / folders
  - Shared mailboxes
  - Calendars
  - Internal docs (Google Drive/SharePoint)
  - Vendor accounts tied to that user

Then execute access transfer.

---

## 10) Final verification (don’t skip)
Confirm:
- User cannot sign in to M365
- User account is disabled in AD
- Mailbox + OneDrive preserved/transferred
- Open tickets re-assigned
- Device status accounted for

---

## 11) Closeout + documentation (ticket notes)
Ticket must include:
- Date/time offboarding completed
- Actions taken:
  - AD disabled + password reset (if done)
  - M365 sign-in blocked + sessions revoked + MFA reset
  - Mailbox/OneDrive handled (who got access)
  - Groups removed (especially privileged)
  - Device recovered/wiped status
- Any pending follow-ups (device return, data exports, approvals)

---

## Common edge cases (quick guidance)
- **Legal hold / audit required:** do not delete anything; follow retention policy.
- **Shared credentials:** rotate them (Wi-Fi keys, service accounts, vendor logins).
- **VIP user:** coordinate with security/HR; time matters.
