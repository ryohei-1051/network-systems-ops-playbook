# Scenario: DNS Resolves but Server Is Unreachable

## Scenario Summary
A client can resolve the destination hostname, but the server or application is still unreachable.
This means DNS is working at least partially, but the full connection path still needs to be verified.

## Initial Framing

Before assuming the root cause, I would separate the issue into several areas:

- DNS resolution
- Client network configuration
- Routing path
- Firewall or security policy
- Destination port reachability
- Server or application availability

The key point is that resolving a hostname only tells the client the destination IP address. It does not guarantee that the client can reach the server or that the application is listening.

## Clarifying Questions

- Is this issue affecting one client or multiple clients?
- Can the client reach the server by IP address?
- What protocol and port should be used?
- Did this connection work before?
- Were there recent DNS, firewall, routing, server, or application changes?
- Is the issue affecting one destination or multiple destinations?

## Expected Behavior

The client should be able to:

- Resolve the destination FQDN to the correct IP address
- Reach the destination network
- Connect to the required service port
- Receive a valid response from the server or application

## Investigation Flow

### 1. Confirm client network configuration

Purpose: To confirm that the client has valid network settings.

Commands or checks:

```powershell
ipconfig /all
```

Expected evidence:

- Valid client IP address
- Correct subnet mask
- Correct default gateway
- Correct DNS server

### 2. Confirm DNS resolution

Purpose: To confirm that the hostname resolves to the expected IP address.

Commands or checks:
```powershell
nslookup <destination-fqdn>
```
Expected evidence:

- DNS server responds
- The FQDN resolves to the expected IP address
- The result is not stale or pointing to the wrong environment

### 3. Test reachability by IP address

Purpose: To confirm whether the issue is still present when DNS is bypassed.

Commands or checks:
```powershell
ping <destination-ip>
tracert <destination-ip>
```

Expected evidence:
- If IP reachability works but FQDN access fails, DNS or application configuration may still be involved
- If IP reachability fails, the issue may be routing, firewall, or destination-side availability

Note: ICMP may be blocked, so ping failure does not always prove the server is down.

### 4. Test required port reachability
Purpose: To confirm whether the required application port is reachable.

Commands or checks:
```powershell
Test-NetConnection <destination-fqdn> -Port 443
Test-NetConnection <destination-ip> -Port 443
```

Expected evidence:
- If the port test succeeds, the network path and port are likely open
- If the port test fails, the issue may be firewall policy, routing, NAT, or the destination service not listening
- If the FQDN port test fails but the IP port test succeeds, DNS, certificate name, proxy, or application virtual host behavior may be involved

### 5. Check routing and firewall path
Purpose: To identify whether traffic is blocked or routed incorrectly between the client and server.

Checks:
- Client route table
- Default gateway
- Firewall policy
- Security group or ACL
- NAT rule if applicable
- Return path from server to client

Expected evidence:
- Correct route exists
- Firewall policy allows the required traffic
- Return traffic is not blocked
- No asymmetric routing issue is present

### 6. Check destination server or service
Purpose: To confirm whether the destination server is online and listening on the expected port.

Checks:
- Server power/status
- Service status
- Listening port
- Local firewall
- Application logs
- System logs

Expected evidence:
- Server is reachable from the correct network
- Service is running
- Service is listening on the expected port
- Logs show successful or failed connection attempts

## Possible Root Causes

| Area        | Possible Cause                                 | Evidence                                            |
| ----------- | ---------------------------------------------- | --------------------------------------------------- |
| DNS         | Hostname resolves to the wrong IP address      | `nslookup` returns unexpected IP                    |
| Client      | Wrong gateway, subnet, or local firewall issue | Client config or local firewall blocks traffic      |
| Routing     | Missing or incorrect route                     | Traceroute stops before destination network         |
| Firewall    | Required port is blocked                       | Port test fails but routing appears correct         |
| NAT         | NAT rule is missing or incorrect               | Traffic reaches firewall but not destination        |
| Server      | Server is down or unreachable                  | No response from destination                        |
| Service     | Application is not listening                   | Server is up but required port fails                |
| Return Path | Server cannot return traffic to client         | Request reaches server but response does not return |

## Verification

The issue is resolved when:

- The FQDN resolves to the correct IP address
- The client can reach the destination network
- The required port is reachable
- The application responds successfully
- Logs confirm successful connection
- The root cause area is documented

## Plain-Language Summary

DNS works like a directory. It tells the computer which IP address belongs to a server name. However, knowing the address does not guarantee that the path is open or that the server application is working. For example, even if a computer knows the correct destination address, traffic can still fail because of a routing issue, firewall rule, blocked port, or server-side service problem. So after confirming DNS, I would continue checking the network path, required port, firewall/security policy, and destination service.

## Reflection

One assumption that could be wrong is that successful DNS resolution means the connection should work. DNS only confirms name-to-IP translation, not full application reachability.

The most useful evidence would be:

- DNS result
- Destination IP reachability
- Required port reachability
- Traceroute result
- Firewall or security policy logs
- Destination service status

The troubleshooting result should document:

- Source client
- Destination FQDN and IP address
- Required protocol and port
- DNS result
- Reachability test result
- Port test result
- Firewall or routing findings
- Final root cause area

To make troubleshooting faster next time, I would follow a consistent order:

1. Confirm client network configuration
2. Confirm DNS result
3. Test destination IP reachability
4. Test required port reachability
5. Check routing and firewall path
6. Check destination server or service
