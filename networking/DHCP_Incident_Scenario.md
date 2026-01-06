**Scenario:** Users can’t get an IP / intermittent connectivity. DHCP clients show APIPA `169.254.x.x`, or leases fail because the scope is exhausted.  
**Goal:** Restore reliable leasing fast, prevent recurrence, and document clearly.

---

## 0) Quick Triage (2–5 min)

### Symptoms you’ll hear/see
- “No internet / limited connectivity”
- Client gets `169.254.x.x` (APIPA)
- `ipconfig /renew` fails
- DHCP server alerts / event logs mention scope exhaustion

### On an affected client (fast confirmation)
Run:
- `ipconfig /all`
  - **Bad:** IPv4 is `169.254.x.x`, **DHCP Enabled: Yes**, no valid gateway
  - **Good:** Valid IPv4 in your subnet, correct gateway, DHCP server listed
- `ipconfig /release` then `ipconfig /renew`

If renewal fails, move to server checks.

---

## 1) Server-Side Confirmations (the “prove it” checks)

### A) Verify the DHCP service is running
On DHCP server:
- Open **Services** → **DHCP Server** → Running
- Or PowerShell:
  - `Get-Service dhcpserver`

### B) Check the scope utilization (is it actually full?)
Open:
- **DHCP console** → IPv4 → **your scope**  
Look at:
- **Address Pool**
- **Address Leases**
- **Scope Statistics** (best view)

PowerShell:
- `Get-DhcpServerv4Scope -ComputerName <DHCP-SERVER>`
- `Get-DhcpServerv4ScopeStatistics -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP>`

**Bad:** Available addresses ~ 0  
**Good:** Healthy buffer (rule of thumb: at least 10–20% free in busy scopes)

### C) Check for conflicts / duplicate DHCP servers
- Confirm **only one DHCP server** is active per VLAN/subnet
- If domain environment: check DHCP server is **authorized** in AD:
  - DHCP console → right click server → **Properties**
  - PowerShell:
    - `Get-DhcpServerInDC`

---

## 2) Immediate Restore Options (choose the safest fast fix)

### Option 1 (Safest): Free addresses by cleaning stale leases
Use when:
- You suspect lots of old/stale devices (phones, guests, laptops)  
Actions:
1. DHCP console → Scope → **Address Leases**
2. Sort by **Lease Expiration**
3. Remove obvious stale entries if appropriate (especially long-expired)

PowerShell (examples):
- List leases:
  - `Get-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP>`
- Remove a specific lease:
  - `Remove-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP> -IPAddress <IP>`

**Be careful:** Don’t delete active critical servers/printers unless you’re sure.

---

### Option 2 (Fast): Reduce DHCP lease duration temporarily
Use when:
- Scope is bursting due to high churn and you need addresses to recycle faster  
In DHCP console:
- Scope → **Properties** → **Lease Duration**
- Temporarily set to something like **4–8 hours** (or **1 day** depending on environment)

**After incident:** return to normal (often 1–7 days depending on org).

---

### Option 3 (Often needed): Expand the DHCP scope / add a new scope
Use when:
- You’ve truly outgrown the subnet (too many devices for /24, etc.)  
Actions:
- Add more IPs to the pool if subnet mask allows
- Or plan subnet expansion (e.g., move from /24 → /23) **requires network change**
- Or create new VLAN/subnet and migrate users (best long-term)

**Reality check:**
- If you’re on a `192.168.10.0/24` and it’s full, you can’t “invent” more addresses without changing the subnet design.

---

### Option 4: Identify and reserve/relocate “always-on” devices
Use when:
- Printers/VoIP/IoT eat addresses and should be controlled  
Actions:
- Convert critical devices to:
  - **DHCP reservation**, or
  - **static outside DHCP range**
- Ensure DHCP range excludes those fixed IPs

---

## 3) Find the Root Cause (why did it fill up?)

### A) Too many devices (growth)
- New hires, new printers, VoIP phones, Wi-Fi expansion, BYOD

### B) Lease duration too long for a high-churn environment
- Guests and mobile devices rotate, but leases don’t expire quickly enough

### C) Rogue DHCP server
- A random router, wireless extender, misconfigured VM handing out leases
Clues:
- Clients show **wrong default gateway**
- DHCP “Server Identifier” is not your DHCP server
Client check:
- `ipconfig /all` → “DHCP Server” field

### D) Multiple scopes/VLAN mismap
- Wrong scope servicing the wrong VLAN  
- Relay (IP helper) misconfigured

---

## 4) Verification (confirm the fix worked)

### On server:
- Scope Statistics shows available addresses > 0
- Event Viewer: no new scope exhaustion errors
- DHCP service healthy

### On 2–3 affected clients:
- `ipconfig /release` + `ipconfig /renew`
- `ipconfig /all` shows:
  - Valid IPv4 in correct subnet
  - Correct gateway/DNS
  - DHCP Server = expected server
- `ping <gateway>`
- `nslookup google.com` (or internal domain)
- Open a web page / access internal resource

---

## 5) Post-Incident Hardening (so it doesn’t happen again)

### Recommended “good” DHCP hygiene
- Maintain **10–20% free** addresses in a scope (buffer)
- Lease duration tuned to environment:
  - Office desktops: 3–7 days often fine
  - High-churn Wi-Fi/guest: shorter (hours–1 day)
- Reserve/static critical devices (printers, servers, VoIP, APs)
- Document:
  - Scope ranges
  - Exclusions
  - Reservations
  - VLAN/subnet mapping

### Monitoring ideas (simple but effective)
- Weekly check scope utilization
- Alert when utilization hits 80–90%
- Track new device growth on Wi-Fi

---

## 6) Documentation Template (copy/paste for ticket)

**Issue:** DHCP scope exhaustion causing clients to fail leasing IP addresses  
**Impact:** X users impacted, unable to access network resources  
**Detection:** Client APIPA `169.254.x.x`; DHCP scope utilization at __%  
**Actions Taken:**
- Verified DHCP service running
- Checked scope stats: available addresses = __
- Cleaned stale leases / adjusted lease duration / expanded scope (details)
**Result:** Clients can renew and receive valid addresses; connectivity restored  
**Prevention:** Adjusted lease duration back to __; added monitoring; planned subnet expansion / created reservations for critical devices  
**Timestamp:** Start __ / End __  
**Owner:** __

---

## 7) Handy Commands (Windows)

### Client
- `ipconfig /all`
- `ipconfig /release`
- `ipconfig /renew`
- `ipconfig /flushdns`
- `nslookup <domain>`
- `ping <gateway>`

### Server (PowerShell)
- `Get-Service dhcpserver`
- `Get-DhcpServerv4Scope -ComputerName <DHCP-SERVER>`
- `Get-DhcpServerv4ScopeStatistics -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP>`
- `Get-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP>`
- `Remove-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-IP> -IPAddress <IP>`
- `Get-DhcpServerInDC`

---

## Quick Rule-of-Thumb Decisions
- **If scope is truly full:** Free stale leases + shorten lease duration (temp), then plan expansion
- **If gateway/DHCP server looks wrong on clients:** suspect rogue DHCP immediately
- **If this keeps happening monthly:** subnet/VLAN redesign is required (capacity issue)
