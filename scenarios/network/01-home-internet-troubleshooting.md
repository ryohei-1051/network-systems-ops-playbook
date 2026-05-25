# Scenario: Home Internet Troubleshooting
## Scenario Summary
You cannot access the Internet from home. How would you troubleshoot?

## Initial Framing
- Scope
Client device
IP configuration (Default gateway), DNS
Internet reachability to specific website/service
ISP or modem/router issue

- Expected behavior
IP configuration should be assigned by DHCP and the IP is valid (not APIPA)
Known websites/services should be reachable
ISP or modem/router works correctly

- Possible affected layers
Layer 3 and Layer 4 or higher

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
Purpose: to check correct configuration
Commands or checks: ipconfig (Windows), ip addr (Linux)
Expected evidence: recive client IP and default gateway

### 2. Check name resolution
Purpose: to see if the client obtain correct destination IP from FQDN
Commands or checks: nslookup (windows), dig (linux)
Expected evidence: there is correct IPs on these commands result

### 3. Check network path

Purpose: To confirm whether the client can reach the local gateway and the Internet.
Commands or checks:
```powershell
ping <default-gateway>
ping 8.8.8.8
tracert 8.8.8.8
```
Expected evidence:
- The client can reach the default gateway
- The client can reach a public IP address
- If DNS fails but public IP works, the issue is likely DNS-related
- If the default gateway fails, the issue is likely local network/client/router related

### 4. Check destination or service side

Purpose: To confirm whether the issue affects all Internet access or only a specific website/service.
Commands or checks:
```powershell
Test-NetConnection google.com -Port 443
Test-NetConnection <specific-website-or-service> -Port 443
```
Expected evidence:
- If multiple websites fail, the issue is likely local network, DNS, router, or ISP-related
- If only one website fails, the issue may be related to the destination service, DNS record, firewall, or application-side problem
- If local checks pass but all Internet access fails, check modem/router status or ISP service status

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
