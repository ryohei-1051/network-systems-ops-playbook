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
IP configuration should be configure by DHCP and the IP is valid (not APIPA)
All the website/service should be reachable
ISP or modem/router works correctly

- Possible affected layers
Layer 3 and Layer 4 or higher

## Clarifying Questions

- Whether the network adapter is configured as static or dynamic
- Is the client can recieve correct IP
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
Commands or checks: ipconfig (Windows), ifconfig/ip -a (Linux)
Expected evidence: recive client IP and default gateway

### 2. Check name resolution
Purpose: to see if the client obtain correct destination IP from FQDN
Commands or checks: nslookup (windows), dig (linux)
Expected evidence: there is correct IPs on these commands result

### 3. Check network path
Purpose: to check gateway device can forward the request from the device
Commands or checks: physically check the modem power light
Expected evidence: the light should be on

### 4. Check destination or service side
Purpose: To check if the traffic have public IPs 
Commands or checks: reach out to the serive provider to check if the account is alive.
Expected evidence: the account should be active

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
When the computer cannot connect to the Internet, it probably dose not know where it is or where it should go. To connect the internet, it should use 2 basic things: source and destination address like when you send a mail via canada post. Regarding source information, we use ipconfig command, for destination, we use nslookup command. With both information, the computer can send a request to the Internet. Also, if the path is not available, the requrest is also unreachable and turns out to be internet failure. To make sure the path is active, we can check modem and ISP.



- What assumption could have been wrong?
- What evidence was most useful?
- What should be documented?
- What would make the troubleshooting faster next time?
