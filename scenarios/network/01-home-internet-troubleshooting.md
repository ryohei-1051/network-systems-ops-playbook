# Scenario: Home Internet Troubleshooting
## Scenario Summary
You cannot access the Internet from home. How would you troubleshoot?

## Initial Framing
- Scope
Client device
IP configuration (Default gateway, DNS)
Internet reachability to specific website/service
ISP or modem/router issue

- Expected behavior
IP configuration should be assigned by DHCP and the IP is valid (not APIPA)
Known websites/services should be reachable
ISP or modem/router works correctly

- Possible affected layers
Layer 1/2: cable, Wi-Fi, adapter, modem/router link  
Layer 3: IP address, default gateway, routing  
Layer 4: TCP/UDP port reachability  
Layer 7: DNS, browser, website/application behavior

## Clarifying Questions

- Whether the network adapter is configured as static or dynamic
- Is the client can receive correct IP
- Is there Internet reachability from the client or only web browser
- Whether the modem powered on or off, or the ISP has some issue at a moment

## Expected Behavior
- Configuration is correct
- The internet site reply with both IP and FQDN
- The modem works correctly (They are not down)
- The account from ISP works correctly (The account is active)

## Investigation Flow

### 1. Check the client side
Purpose: To confirm whether the client has valid network configuration.
Commands or checks:
```powershell
ipconfig /all
```
Linux:
```bash
ip addr
ip route
```
Expected evidence:
- The network adapter is enabled
- The client has a valid IP address
- The client is not using an APIPA address such as 169.254.x.x
- The client has a default gateway
- The client has DNS servers configured

### 2. Check local gateway reachability
Purpose: To confirm whether the client can reach the local router/default gateway.
Commands or checks:
```powershell
ping <default-gateway>
```
Expected evidence:
- If the default gateway replies, the local network path is working
- If the default gateway does not reply, the issue may be client-side, Wi-Fi/Ethernet, router, or local network related

### 3. Check public IP reachability
Purpose: To confirm whether the client can reach the Internet without depending on DNS.
Commands or checks:
```powershell
ping 8.8.8.8
tracert 8.8.8.8
```
Expected evidence:
- If public IP works but websites fail, DNS or browser/application issues become more likely
- If public IP fails, the issue may be router, modem, ISP, routing, or upstream connectivity

### 4. Check name resolution
Purpose: To confirm whether the client can translate a website name into an IP address.
Commands or checks:
```powershell
nslookup google.com
```
Linux:
```bash
dig google.com
```
Expected evidence:
- DNS server responds
- The FQDN resolves to valid IP addresses
- If DNS fails but public IP works, the issue is likely DNS-related

### 5. Check destination or service side
Purpose: To confirm whether the issue affects all Internet access or only a specific website/service.
Commands or checks:
```powershell
Test-NetConnection google.com -Port 443
Test-NetConnection <specific-website-or-service> -Port 443
```
Expected evidence:
- If multiple websites fail, the issue is likely local network, DNS, router, modem, or ISP-related
- If only one website fails, the issue may be related to the destination service, DNS record, firewall, or application-side problem

### 6. Check modem/router or ISP status
Purpose: To confirm whether the home network edge device or ISP service is causing the issue.
Commands or checks:
- Check modem/router power and status lights
- Reboot modem/router if appropriate
- Check ISP outage page or contact ISP support
- Confirm the ISP account is active
Expected evidence:
- Modem/router is powered on
- WAN/Internet status is active
- ISP has no outage or account issue

## Possible Root Causes

|   Area   |       Possible Cause       |    Evidence    |
|----------|----------------------------|----------------|
|  Client  | network adapter is disable | command result |
|   DNS   |   no access to DNS server  | command result |
| Network |    modem has some issue    | device check |
| Service | contract or ISP issue | account activation check |

## Verification

The issue is resolved when:
- there is correct IP configured on network adapter 
- the client has destination IPs from FQDN
- ping reachability to the destination server
- modem turned on / ISP account is active

## Plain-Language Summary
When a computer cannot connect to the Internet, I would first check whether the computer has the correct address information and knows where to send traffic. This is similar to sending mail. The computer needs its own address, a destination address, and a path to send the traffic. The `ipconfig` command helps confirm the computer's own network information, such as its IP address, default gateway, and DNS server. The `nslookup` command helps confirm whether a website name can be translated into an IP address. If the computer has the correct address information but the Internet still does not work, the next step is to check the path. This includes testing the default gateway, testing a public IP address, checking DNS, and confirming whether the modem/router or ISP has an issue.

## Reflection
One assumption that could be wrong is that the issue is caused by the ISP or modem. The problem may actually be limited to one client device, such as a disabled adapter, wrong IP configuration, DNS issue, or browser-specific problem. The most useful evidence would be the client IP configuration, default gateway reachability, DNS resolution result, and whether the client can reach a public IP address such as 8.8.8.8. The troubleshooting result should document the affected device, IP address, DNS server, gateway test result, modem/router status, and whether the issue was client-side, local network-side, or ISP-side. To make troubleshooting faster next time, I would follow a consistent order: check the client IP configuration first, then test the default gateway, then test public IP reachability, then test DNS, and finally check the modem/router or ISP status.
