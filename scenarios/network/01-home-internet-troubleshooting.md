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
- Can the client receive the correct IP address?
- Is there Internet reachability from the client or only web browser
- Whether the modem is powered on or off, or the ISP has some issue at a moment

## Expected Behavior
- Configuration is correct
- Internet sites should be reachable by both IP address and FQDN
- The modem works correctly (It is not down)
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

| Area | Possible Cause | Evidence |
|---|---|---|
| Client | Network adapter is disabled or misconfigured | No valid IP address, no default gateway, or APIPA address |
| Local Network | Client cannot reach the default gateway | Ping to default gateway fails |
| DNS | DNS server is unreachable or returns incorrect results | `nslookup` fails or returns unexpected IP |
| Router/Modem | Home router or modem has an issue | No WAN status, warning lights, or gateway unreachable |
| ISP | ISP outage or inactive account | Multiple devices fail and ISP confirms outage/account issue |
| Destination Service | Only one website or service is unavailable | Other websites work, but specific service fails |

## Verification

The issue is resolved when:

- The client has a valid IP address, default gateway, and DNS server
- The client can reach the default gateway
- The client can reach a public IP address
- The client can resolve FQDNs using DNS
- Required websites or services are reachable on the expected port, such as TCP 443
- The result confirms whether the issue was client-side, local network-side, ISP-side, or destination-side

## Plain-Language Summary
When a computer cannot connect to the Internet, I would first check whether the computer has the correct address information and knows where to send traffic. This is similar to sending mail. The computer needs its own address, a destination address, and a path to send the traffic. The `ipconfig` command helps confirm the computer's own network information, such as its IP address, default gateway, and DNS server. The `nslookup` command helps confirm whether a website name can be translated into an IP address. If the computer has the correct address information but the Internet still does not work, the next step is to check the path. This includes testing the default gateway, testing a public IP address, checking DNS, and confirming whether the modem/router or ISP has an issue.

## Reflection

One assumption that could be wrong is that the issue is caused by the ISP or modem. The problem may actually be limited to one client device, such as a disabled adapter, wrong IP configuration, DNS issue, or browser-specific problem.

The most useful evidence would be:

- Client IP configuration
- Default gateway reachability
- Public IP reachability
- DNS resolution result
- Specific service port reachability

The troubleshooting result should document:

- Affected device
- IP address
- Default gateway
- DNS server
- Gateway test result
- Public IP test result
- DNS test result
- Modem/router status
- Final root cause area

To make troubleshooting faster next time, I would follow a consistent order:

1. Check client IP configuration
2. Test the default gateway
3. Test public IP reachability
4. Test DNS
5. Test the specific website or service
6. Check modem/router or ISP status
