# Goal

Get the new hire productive Day 1, with proper access, security, and clean documentation.

## Step 1 — Intake checklist (from HR/manager)

You need:

Full name (legal + preferred)

Start date + timezone/office

Role + department

Manager name

Required access:

Email distribution lists

Shared drives

Apps (CRM, ticketing, M365, etc.)

Any special groups (Finance, Admin, etc.)

Hardware needs:

Laptop/desktop

Dock/monitors

Headset

Security keys (if used)

## Step 2 — Create the AD user (ADUC)

Open ADUC → navigate to the correct Users OU

Right-click OU → New → User

Fill in:

First / Last

Username (follow your naming convention)

Set temp password

Check User must change password at next logon

Finish

Then:

Open user Properties

Add details (optional but nice):

Title, Department, Manager

Office, phone (if you use it)

## Step 3 — Add group memberships (access)

In user Properties → Member Of

Add baseline groups (example):

All Employees

Department group (Sales/Operations/etc.)

Shared drive access groups

Email distribution lists (if group-based)

Best practice: use role-based groups (not manual folder permissions per user).

## Step 4 — Create mailbox / M365 account (GUI flow depends on setup)

This varies by org (Hybrid/Entra ID). In many environments you’ll:

Create in AD → sync to M365 → assign license in Admin Center

Checklist:

Ensure they can sign into email

Ensure MFA enrollment (if required)

## Step 5 — Provision the computer
If using Autopilot/Intune (modern)

Assign device to user

Confirm compliance policies

Required apps deploy automatically

If manual imaging (classic)

Image PC / install OS

Join domain (or enroll in Intune)

Ensure hostname naming standard

Install baseline apps:

Browser, PDF reader

Office apps

Remote support agent

Security agent

Printer mapping (if needed)

## Step 6 — Verify core access (don’t skip)

Log in with user (or have them log in) and validate:

Email works

Shared drives open

Key apps sign in

Printers appear (if needed)

VPN works (if remote)

This is where you catch missing groups early.

## Step 7 — Onboarding handoff (quick user orientation)

Give them:

How to contact IT (portal/email/phone)

Password/MFA expectations

Common fixes:

reboot

VPN rules

where to find guides/runbooks

## Step 8 — Document everything (ticket closeout)

Include:

Account created (username)

Groups added

Hardware asset tag + assigned device name

Licenses assigned (if applicable)

Apps installed / enrollment method

Any pending requests/escalations
