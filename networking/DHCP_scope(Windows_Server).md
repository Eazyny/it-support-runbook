## 0) Quick “is this DHCP?” check (client-side)

On an affected PC, confirm whether it’s failing to get a lease.

ipconfig /all


What “good” looks like:

IPv4 Address is in your expected subnet (ex: 10.10.10.x)

Default Gateway is present (ex: 10.10.10.1)

DHCP Enabled: Yes

DHCP Server: shows your DHCP server IP

Lease Obtained/Expires look normal

Red flags:

IPv4 Address is 169.254.x.x (APIPA)

No Default Gateway

DHCP Server field missing

Try renew:

ipconfig /release
ipconfig /renew


What “good” looks like:

Renew succeeds quickly and you get a valid IP + gateway.

## 1) Confirm DHCP service is up (server-side)

On the DHCP server:

Get-Service dhcpserver


What “good” looks like:

Status = Running

If it’s not running (or acting weird), restart:

Restart-Service dhcpserver

## 2) Identify the scope + check if it’s actually exhausted

List scopes:

Get-DhcpServerv4Scope -ComputerName <DHCP-SERVER>


Then check scope stats:

Get-DhcpServerv4ScopeStatistics -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID>


What “good” looks like:

Free addresses > 10–20 (depends on environment)

% In Use not pinned at 95–100%

Red flags:

Free = 0 or very low

In Use ~ 99–100%

## 3) Look for what’s eating the pool (leases)

Pull leases for the scope:

Get-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID>


If you want to see who expires soonest / spot stale patterns:

Get-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> | Sort-Object LeaseExpiryTime


What “good” looks like:

Lease list roughly matches expected device count.

You recognize vendor types / hostnames (domain PCs, printers, etc).

Red flags:

Huge spike in unknown devices

Tons of random hostnames / MACs (guest phones, IoT explosion)

Leases that look ancient or never rotating

## 4) Fast “stop the bleeding” fix (temporary): shorten lease duration

This helps addresses recycle faster without deleting anything.

First, view current scope settings:

Get-DhcpServerv4Scope -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> | Select-Object ScopeId,StartRange,EndRange,SubnetMask,State,LeaseDuration


Then temporarily shorten (example: 4 hours):

Set-DhcpServerv4Scope -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -LeaseDuration 04:00:00


What “good” looks like:

New clients start getting leases again after a bit.

Pool slowly recovers as old leases churn.

Note: Later, set it back to something sane (like 8–24 hours depending on your environment).

## 5) Free up IPs: remove clearly stale leases (carefully)

Only do this if you’re confident the leases are not active.
Start by filtering to expired leases if you’re tracking those, otherwise you’ll do this based on LeaseExpiryTime patterns.

(If you tell me what your leases look like, I’ll tell you the safest filter.)

Remove a specific lease:

Remove-DhcpServerv4Lease -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -IPAddress <IP>


What “good” looks like:

You free some space.

The next affected client can renew successfully.

## 6) Expand capacity: widen the available range (if subnet allows)

First, confirm your start/end range and subnet mask:

Get-DhcpServerv4Scope -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> | Select-Object ScopeId,StartRange,EndRange,SubnetMask


Then check exclusion ranges (sometimes exclusions are accidentally huge):

Get-DhcpServerv4ExclusionRange -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID>


If exclusions are wrong, adjust them:

Remove-DhcpServerv4ExclusionRange -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -StartRange <START-IP> -EndRange <END-IP>


If your scope range can be expanded (and you have free IP space in that subnet), you’d typically adjust scope start/end in the GUI. (PowerShell can do it too, but the cmdlets vary depending on how your scope is defined — so I keep this part “safe” unless you want the exact command for your server version.)

## 7) Validate DHCP options (gateway/DNS) so leases are usable

Check scope options:

Get-DhcpServerv4OptionValue -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID>


Common ones:

Router (Default Gateway)

DNS Servers

DNS Domain

Set them if needed:

Set-DhcpServerv4OptionValue -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -Router <GATEWAY-IP>
Set-DhcpServerv4OptionValue -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -DnsServer <DNS1>,<DNS2>
Set-DhcpServerv4OptionValue -ComputerName <DHCP-SERVER> -ScopeId <SCOPE-ID> -DnsDomain <DOMAIN>


What “good” looks like:

Clients can resolve internal names and reach the gateway right after renewal.

## 8) Confirm the fix on 2–3 clients

On clients:

ipconfig /renew
ipconfig /all
ping <default-gateway>
nslookup <internal-hostname>


What “good” looks like:

Valid IPv4 + gateway

Ping gateway works

nslookup works
