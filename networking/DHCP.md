````md
# DHCP Runbook (IT Support / Help Desk)

## Purpose
Restore network connectivity when a device can’t get an IP address (or gets the wrong one), troubleshoot DHCP scope issues, and verify DHCP handoff from router/firewall to clients.

## Symptoms (What users report)
- “No internet” / “Connected, no internet”
- “Limited connectivity”
- Can’t reach internal resources (file share, printer, VPN, intranet)
- Works on Wi-Fi but not Ethernet (or vice versa)

## Fast facts (What DHCP does)
DHCP automatically assigns:
- IP Address
- Subnet Mask
- Default Gateway
- DNS Servers
- Lease duration + renewal behavior

If DHCP fails, clients may self-assign **169.254.x.x (APIPA)** → classic “no DHCP response” sign.

---

## 0) Quick Triage (60 seconds)

### A) Confirm the basics
- Is it **only 1 user** or **many users**?
  - **Many users** = likely DHCP server/scope/network outage
  - **One user** = local NIC/Wi-Fi issue, bad lease, VLAN/port config, etc.

### B) Check the client’s IP
**Windows**
```powershell
ipconfig /all
````

Look for:

* IPv4: **169.254.x.x** = no DHCP lease
* Default Gateway missing = no route off subnet
* DNS pointing wrong = name resolution issues

**macOS**

* System Settings → Network → (Wi-Fi/Ethernet) → Details → TCP/IP

---

## 1) Client-side Fixes (Help Desk core)

### Step 1 — Release/Renew

**Windows**

```powershell
ipconfig /release
ipconfig /renew
```

### Step 2 — Reset network stack (Windows)

```powershell
ipconfig /flushdns
netsh winsock reset
netsh int ip reset
```

Reboot after.

### Step 3 — Disable/Enable adapter

* Settings → Network → Advanced network settings → Disable/Enable
  or
* `ncpa.cpl` → right-click adapter → Disable/Enable

### Step 4 — Try another network path

* If on Wi-Fi, test Ethernet (or phone hotspot)
* This isolates “DHCP on this network/VLAN” vs “client device issue”

---

## 2) Identify What “Network” They’re On (VLAN / SSID)

DHCP is usually per VLAN/subnet.

Ask:

* Which **SSID**? (Corp / Guest / IoT)
* Which **jack/port**? (Desk port, conference room, etc.)
* Was it recently moved to a new desk?

If only one area is broken → check switch port VLAN or AP mapping.

---

## 3) Determine the DHCP Source

Common DHCP sources:

* Router/Firewall (small office)
* Windows DHCP Server
* DHCP service on Linux (dnsmasq/isc-dhcp-server)
* Cloud-managed network (Meraki/UniFi, etc.)

You should know:

* DHCP server IP/name
* Scope/subnet for that network (ex: `10.10.10.0/24`)
* Gateway IP
* DNS server IPs

---

## 4) DHCP Server / Scope Checks (when multiple users affected)

### A) Check scope utilization (IP pool full)

If the scope runs out of free addresses:

* Clients fail to obtain leases
* You’ll see random failures as leases churn

Fix options:

* Increase scope range (expand pool)
* Reduce lease duration temporarily (careful)
* Identify rogue/static IPs squatting in range
* Remove old/stale leases if appropriate

### B) Check DHCP service health

If Windows DHCP server:

* Confirm service running: **DHCP Server**
* Check Event Viewer → DHCP Server logs
* Verify server can reach that VLAN (routing present)

### C) Verify DHCP relay (if server isn’t in same VLAN)

If DHCP server is on another subnet, the network needs a relay (IP helper).
If relay breaks, that VLAN can’t reach DHCP.

Symptoms:

* One VLAN dead, others fine
* APIPA addresses in that VLAN

Fix is usually on router/L3 switch:

* Add/verify `ip helper-address <DHCP_SERVER_IP>` (Cisco)
* Confirm relay configured on correct SVI/interface

---

## 5) Detect Rogue DHCP (Very important)

Rogue DHCP = another device handing out leases (bad router, hotspot, misconfigured appliance).

Signs:

* Clients get “weird” gateway/DNS
* IP range doesn’t match your expected subnet
* Random redirect/captive portal behavior

**Windows**

```powershell
ipconfig /all
```

Check:

* **DHCP Server**: is it the correct server?
* **Default Gateway**: expected?
* **DNS servers**: expected?

If rogue suspected:

* Escalate immediately (network/security)
* Find MAC/source via switch/AP controller
* Remove unauthorized device

---

## 6) Verification (make sure it’s truly fixed)

After changes:

* Client gets correct IP + gateway + DNS
* Ping gateway:

  * `ping <gateway>`
* DNS resolution:

  * `nslookup google.com`
  * `nslookup internal-hostname` (if applicable)
* Browse to a known site + internal resource if relevant

---

## 7) Common Root Causes + Fix Map

### Client has 169.254.x.x (APIPA)

* DHCP server down → restart service / restore server
* DHCP relay missing/broken → fix helper/relay config
* VLAN wrong on switch port → correct port VLAN
* Wi-Fi SSID mapped wrong VLAN → fix SSID/VLAN mapping
* Cable/port bad → test cable/port/switch

### Client has an IP but no internet

* Gateway missing/wrong → DHCP option issue or rogue DHCP
* DNS wrong → DHCP option issue; set correct DNS servers
* Captive portal/guest network → user on wrong SSID/VLAN

### Some users can connect, others can’t

* Scope nearly full → expand pool / shorten lease (temp)
* Sticky bad leases → clear old lease / release-renew
* Rogue DHCP intermittent

---

## 8) Ticket Notes Template (copy/paste)

When you close the ticket, log:

* User/device:
* Connection type (Wi-Fi/Ethernet):
* SSID / port / location:
* IP before:
* IP after:
* DHCP server expected:
* What was done (steps):
* Root cause:
* Prevention note (if any):

