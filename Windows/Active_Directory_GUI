Active Directory GUI Runbook (Windows Server)
Tools I may use

Active Directory Users and Computers (ADUC)
Server Manager → Tools → Active Directory Users and Computers

(Sometimes) Group Policy Management (GPMC)
Server Manager → Tools → Group Policy Management

(Sometimes) DNS Manager
Server Manager → Tools → DNS

1) Unlock a user (most common ticket)

Scenario: “I’m locked out” / password attempts.

Open ADUC

Find the user:

Either browse the OU or

Right-click the domain → Find… → search by name / username

Right-click user → Properties

Go to Account tab

Check “Unlock account”

Click Apply → OK

What “good” looks like:

User can sign in immediately

If they keep locking out, it’s usually a saved password somewhere (phone mail app, mapped drive, old Outlook creds)

2) Reset a user password

Scenario: “Forgot password” / “Password expired”.

In ADUC, right-click the user

Click Reset Password…

Set temporary password

Check “User must change password at next logon”

Click OK

What “good” looks like:

User signs in with temp password and is forced to change it

If they’re remote, confirm they’re using VPN/SSO flow required by your org

3) Enable / Disable an account
Enable (new hire / mistakenly disabled)

Right-click user

Click Enable Account

Disable (termination / leave of absence)

Right-click user

Click Disable Account

(Best practice) Move to Disabled Users OU:

Drag-and-drop to OU=Disabled Users
or right-click → Move…

What “good” looks like:

Disabled users can’t authenticate

Account is in the correct OU for auditing

4) Add user to a group (grant access)

Scenario: “Needs access to X drive / app / email group”

In ADUC, find user → right-click → Properties

Go to Member Of tab

Click Add…

Type group name → Check Names → OK

Apply → OK

What “good” looks like:

Group appears under Member Of

User logs off/on (or locks/unlocks) and access works

Common gotcha:

Some access changes need a full sign-out/sign-in to refresh the token

5) Remove user from a group (remove access)

User Properties → Member Of

Select group

Click Remove

Apply → OK

What “good” looks like:

Group removed

Access no longer works after next logon/token refresh

6) Check why a user has access (audit-style quick check)

Scenario: “Why can they access this share/app?”

User Properties → Member Of

Look for:

department groups

“All Employees”

role-based groups (Finance, Helpdesk, etc.)

If access is indirect, it’s usually via nested groups:

Double click the group → check Member Of for parent groups

What “good” looks like:

You can trace the access path (user → group → parent group)

7) Move a computer into the correct OU (GPO not applying)

Scenario: “New PC joined domain but didn’t get policies / software”

In ADUC, find the computer object

Often shows up in Computers container by default

Right-click the computer → Move…

Choose correct OU (ex: OU=Workstations,OU=NYC)

Click OK

Have user reboot or run gpupdate (client side)

What “good” looks like:

Computer object lives in the OU where your workstation GPOs are linked

Policies/software begin to apply

8) Create a new user (basic HR request)

Navigate to the correct OU (ex: OU=Users)

Right-click OU → New → User

Fill:

First name, Last name

User logon name (UPN)

Set temp password

Check:

User must change password at next logon

Finish

Add baseline groups:

User Properties → Member Of → Add…

What “good” looks like:

User is created in the correct OU

Baseline group membership is correct on day 1

9) GPO (GUI) basics: confirm a policy is linked where you think it is

Scenario: “Policy isn’t applying” and you want to sanity check.

Open Group Policy Management

Expand your domain → browse OUs

Click the OU

In the right pane, check Linked Group Policy Objects

What “good” looks like:

The correct GPO is linked to the correct OU

Link order makes sense (top = highest priority)

Common gotchas:

Policy linked to wrong OU

Security filtering excludes the user/computer

GPO disabled (either User/Computer side disabled)

10) DNS quick GUI check (when logins/AD discovery is weird)

Scenario: Random sign-in issues, “domain not available”, GPO failures.

Open DNS Manager

Check your domain forward lookup zone

Confirm SRV records exist:

_msdcs

_ldap

_kerberos

Confirm DCs appear in the zone

What “good” looks like:

SRV records exist

DC hostnames resolve inside DNS

The “Helpdesk AD flow” you’ll use constantly

Find user → check locked/disabled

Unlock/reset password

Check group membership (access)

If device/policy issue → verify computer OU

If AD-wide weirdness → check DC/DNS/replication (usually escalated)
