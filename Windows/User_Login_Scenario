Goal

Get the user back in fast, while figuring out why it happened so it doesn’t repeat.

First 60 seconds: triage questions (save time)

Ask (or infer) these quickly:

Are you on-site or remote (VPN)?

Is this a new device, password change, or first login?

What exactly do you see?

“Password incorrect”

“Account locked”

“Your account has been disabled”

“Domain not available”

“The trust relationship failed”

“We can’t sign you in” (profile)

Can they log in on another device?

These answers determine which branch you take.

Step 1 — Confirm the identity + username

Why: Half of login issues are “wrong account / wrong domain / wrong username”.

Confirm the login format they’re using:

DOMAIN\username or username@company.com

If they have multiple accounts (common with contractors), make sure they’re on the right one.

Step 2 — Check the AD account status (ADUC)

Open: Server Manager → Tools → Active Directory Users and Computers

Find the user (Right-click domain → Find…)

Open Properties

Account tab:

Is Account is locked out showing? (or checkbox “Unlock account” available)

Is the account disabled?

Any restrictions like Logon Hours or Log On To… (rare, but it exists)

If locked out

Check Unlock account

Click Apply → OK

Tell user to try again

If it re-locks quickly → jump to Step 6 (Repeated lockouts)

If disabled

Right-click user → Enable Account

Confirm if this was intentional (HR/manager)

Then try login again

Step 3 — Password / Expired password path

Symptoms:

“Password incorrect” but user “swears it’s right”

“Your password has expired”

They recently changed password but “old password works somewhere else”

In ADUC:

Right-click user → Reset Password

Set temp password

Check User must change password at next logon

OK

What to tell user:

Use the temp password once

Immediately set a new password

If remote: they may need VPN or the right reset portal depending on your org.

Step 4 — “Domain not available” / can’t reach domain (network path)

Symptoms:

“The domain is not available”

Login screen takes forever

Works on-site but not at home

Do this first:

Confirm user has internet if remote

Confirm VPN is connected (if required before login)

Quick Windows checks (GUI):

System tray network icon shows connected

If Wi-Fi: toggle Wi-Fi off/on, reconnect

If Ethernet: link light, try another cable/port

If on a domain-joined laptop off-site:

Some orgs require VPN before domain auth for first login after password reset

Workaround: log in using cached credentials if possible, then VPN, then change password.

Step 5 — Computer object / OU / device issues (ADUC)

Symptoms:

User can’t login on one PC, but can on another

“Trust relationship failed”

PC feels “not in domain”

In ADUC:

Find the computer object

Confirm:

It exists

It’s in the correct OU

It’s not disabled

“Trust relationship failed” (common fix)

This usually becomes a rejoin domain fix (often escalated), but your checklist is:

Verify PC has correct DNS pointing to DC (usually escalated)

If approved: remove from domain → reboot → rejoin domain → reboot

(If you want, I’ll make a separate mini-runbook for this one.)

Step 6 — Repeated lockouts (they unlock, then lock again)

Symptoms: User gets unlocked, tries once, locks again.

Most common causes:

Phone email app using old password

Outlook / Teams / OneDrive cached creds

Saved Windows credentials (Credential Manager)

Mapped drives / printers using old creds

Another device still using old password

Your move:

Ask: “Did you change your password recently?”

Have them update password on:

Phone mail app

Any second laptop

Outlook/Teams sign-in

If they’re on Windows:

Control Panel → Credential Manager → remove old entries (carefully)

Then unlock again and test.

Step 7 — Document + close properly

Ticket notes should include:

Symptom/error message

What you checked (locked/disabled/password reset/VPN)

What you changed (unlock/reset/groups)

Root cause (if found)

What user confirmed (success + time)
