## 0) Fast triage (what am I seeing?)

### Windows (client)
**Commands**
- `ipconfig /all`
- `ipconfig /release`
- `ipconfig /renew`
- `ipconfig /flushdns`

**What “good” looks like**
- IPv4 address is in the expected subnet (ex: `192.168.1.x`)
- **Default Gateway** matches your router/L3 gateway (ex: `192.168.1.1`)
- **DHCP Enabled** = `Yes`
- **DHCP Server** shows the correct server/router IP (ex: `192.168.1.1` or `10.10.10.10`)
- **DNS Servers** are expected (router, internal DNS, or public like `1.1.1.1/8.8.8.8`)
- **Lease Obtained/Expires** updates after renew (time changes)

**Bad signs**
- `169.254.x.x` (APIPA) = didn’t get a lease
- Gateway missing or wrong
- DNS points to something random/unexpected
- DHCP Server is not your expected device (possible rogue DHCP)

---

## 1) Confirm connectivity to the network path

### Windows
**Commands**
- `ping <default-gateway>`
- `arp -a`
- `tracert 8.8.8.8`

**What “good” looks like**
- Ping to gateway succeeds (low ms on LAN)
- `arp -a` shows a MAC for gateway IP
- `tracert 8.8.8.8` shows first hop = gateway

**Bad signs**
- Can’t ping gateway → Wi-Fi/VLAN/cabling/switch port issue
- ARP table empty or gateway missing → L2/L3 issue

---

## 2) Quick DHCP client service health (Windows)

**Commands**
- GUI: `services.msc` → **DHCP Client** service
- PowerShell: `Get-Service Dhcp`
- Restart service:
  - `net stop dhcp`
  - `net start dhcp`

**What “good” looks like**
- DHCP Client service is **Running**
- Renew works after service restart

**Bad signs**
- Service won’t start or stops immediately → system corruption, policy restriction, or deeper OS issue

---

## 3) Check interface + Wi-Fi basics

### Windows
**Commands**
- `netsh interface ip show config`
- `netsh wlan show interfaces`
- `netsh wlan show profiles`
- Disable/enable adapter quickly:
  - `ipconfig /release`
  - Disable/Enable in Network Settings (or Device Manager)

**What “good” looks like**
- Correct SSID, strong signal (for Wi-Fi)
- Adapter has DHCP enabled
- No “Media disconnected” unless unplugged/no Wi-Fi

---

## 4) Validate DNS after getting a lease (don’t confuse DHCP vs DNS)

**Commands**
- `nslookup google.com`
- `nslookup <internal-hostname>` (if domain environment)
- `ipconfig /displaydns` (optional)

**What “good” looks like**
- `nslookup` returns IPs quickly
- Internal names resolve to internal IPs (if you have internal DNS)

**Bad signs**
- IP lease looks correct but DNS fails → DNS issue, not DHCP
- DNS servers are wrong → DHCP option misconfigured (Option 6)

---

## 5) “I got an IP, but internet is dead” (gateway/DNS checks)

**Commands**
- `ping 8.8.8.8`
- `ping google.com`

**Interpretation**
- `ping 8.8.8.8` works but `ping google.com` fails → DNS issue
- Both fail but you can ping gateway → routing/upstream issue
- Can’t ping gateway → local network issue

---

## 6) DHCP server quick checks (home/small office router)

If DHCP is on your router:
- Reboot router (last resort if production)
- Confirm DHCP is enabled and scope/pool has free leases
- Check for MAC filtering or “static IP reservations” conflicts

**What “good” looks like**
- DHCP enabled
- Pool has available addresses
- Lease time reasonable (e.g., 8–24 hours for office; 24h+ for home)
- DNS/gateway options correct

---

## 7) Windows Server DHCP (basic admin checks)

> Use these if you have access to the Windows DHCP server.

### PowerShell (run on DHCP server)
**Common**
- `Get-DhcpServerv4Scope`
- `Get-DhcpServerv4ScopeStatistics`
- `Get-DhcpServerv4Lease -ScopeId <scope-network>`

**What “good” looks like**
- Scopes are **Active**
- **Free** addresses exist (not at 0)
- Leases are being issued recently
- No obvious runaway device count (like thousands of leases unexpectedly)

### Event Viewer
- **Event Viewer → Applications and Services Logs → Microsoft → Windows → DHCP-Server**
- Look for:
  - Scope full events
  - Authorization issues (in domain environments)
  - Service stop/start events

**What “good” looks like**
- No repeating critical errors
- Lease assignment events normal volume

---

## 8) “Good defaults” checklist (DHCP options)

Typical DHCP options you expect:
- **Router/Gateway (Option 3):** your gateway IP
- **DNS (Option 6):** internal DNS (domain) or router/public DNS (small/home)
- **DNS Domain Name (Option 15):** your internal domain (if applicable)
- **NTP (Option 42):** optional, but consistent if used

**What “good” looks like**
- Clients consistently receive the same gateway + DNS set
- No devices getting different DNS/gateway unless intentionally segmented (VLANs)

---

## 9) Quick “is this rogue DHCP?” suspicion checks

**Client indicators**
- `ipconfig /all` shows **DHCP Server** = unknown IP
- DNS points to unfamiliar addresses
- Gateway is wrong
- Multiple clients show different DHCP server IPs on the same VLAN

**Immediate safe actions**
- Move client to a known-good port/Wi-Fi
- Disconnect suspicious device(s) if identified
- Escalate to network/security response (see `rogue-dhcp-incident.md`)

---

## 10) What to record in the ticket (fast + useful)

- Device + user + location (Wi-Fi SSID or switch port)
- `ipconfig /all` key fields:
  - IP / mask / gateway / DNS / DHCP server
  - Lease obtained/expires
- What changed recently (new switch, AP, router, VLAN changes)
- Test results:
  - ping gateway
  - ping 8.8.8.8
  - nslookup result
- Resolution steps taken (renew, adapter reset, service restart, etc.)

---

## Quick reference: “good” outcomes by symptom

### Symptom: 169.254.x.x (APIPA)
Good fix path:
- Link is up → DHCP Client running → renew succeeds → gets correct IP/gateway/DNS

### Symptom: Has IP but no internet
Good diagnosis:
- Ping 8.8.8.8 fails → routing/upstream
- Ping 8.8.8.8 works but domain fails → DNS

### Symptom: Wrong gateway/DNS
Good diagnosis:
- DHCP option misconfig OR rogue DHCP on the segment
- Compare multiple clients; check DHCP Server field

---
