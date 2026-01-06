# DNS Troubleshooting Checklist (Help Desk)

## Quick Checks
- Confirm the device has an IP address (`ipconfig` / `ip a`)
- Ping the gateway/router
- Ping a public IP (ex: 1.1.1.1) to test internet without DNS
- Try resolving a domain (ex: `nslookup google.com`)

## Windows Commands
- `ipconfig /all`
- `nslookup google.com`
- `ping 1.1.1.1`
- `ipconfig /flushdns`
- `ipconfig /release` then `ipconfig /renew`

## Common Causes
- Wrong DNS server
- Router DNS issues
- VPN/DNS filtering
- Captive portal
- Browser cache vs system DNS confusion

## Verification
- Confirm websites load
- Confirm multiple domains resolve

