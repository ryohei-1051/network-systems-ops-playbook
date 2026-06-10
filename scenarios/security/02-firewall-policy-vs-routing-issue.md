# Scenario: Firewall Policy Versus Routing Issue

## Scenario Summary

A client cannot reach a server or application across the network. The goal is to determine whether the issue is caused by routing, firewall/security policy, NAT, or return-path behavior.

This scenario focuses on separating two common problem areas:

* Routing: whether traffic has a valid path to the destination
* Firewall policy: whether traffic is allowed by security rules

## Initial Framing

Before assuming that the firewall is blocking traffic, I would first confirm the expected communication flow.

The key questions are:

* What is the source IP?
* What is the destination IP or FQDN?
* What protocol and port should be used?
* What network path should the traffic take?
* Is there NAT involved?
* Does the return traffic have a valid path back?

The key point is that a connection failure is not automatically a firewall issue. It may be routing, NAT, DNS, server-side listener, local firewall, asymmetric routing, or return-path failure.

## Plain-Language Explanation

Routing decides where traffic should go.

Firewall policy decides whether traffic is allowed to pass.

A simple analogy is a road and a security gate. Routing is like having a road to the destination. Firewall policy is like having permission to pass through the gate.

If there is no road, the traffic cannot reach the gate. If the road exists but the gate blocks access, then routing may be correct but the firewall policy is denying the traffic.

## Expected Behavior

The client should be able to:

* Resolve the destination name if FQDN is used
* Send traffic toward the correct destination IP
* Follow a valid routing path
* Pass through firewall/security policy
* Reach the required destination port
* Receive return traffic from the server
* Complete the application connection

## Clarifying Questions

* Is the issue affecting one client, one subnet, one site, or multiple users?
* What source and destination IP addresses are involved?
* What protocol and port are required?
* Did this connection work before?
* Were there recent firewall, routing, NAT, DNS, or server changes?
* Is the traffic internal, internet-bound, VPN-based, or across sites?
* Is NAT required?
* Is the server listening on the expected port?
* Are firewall logs available?
* Is the return path symmetric or asymmetric?

## Investigation Flow

### 1. Confirm the traffic flow

Purpose: To define exactly what communication should happen before troubleshooting.

Information to collect:

```text
Source IP:
Destination FQDN:
Destination IP:
Protocol:
Port:
Expected path:
NAT required:
Firewall zones/interfaces:
```

Expected evidence:

* Source and destination are clearly identified
* Required protocol and port are known
* Expected path is understood
* NAT and firewall zones are identified if applicable

### 2. Confirm client-side configuration

Purpose: To make sure the source device has valid network configuration.

Commands or checks:

```powershell
ipconfig /all
route print
```

Linux:

```bash
ip addr
ip route
```

Expected evidence:

* Client has a valid IP address
* Default gateway is configured
* DNS server is configured if FQDN is used
* Route table has a path toward the destination or default route

### 3. Confirm name resolution if FQDN is used

Purpose: To make sure the client is trying to reach the correct destination IP.

Commands or checks:

```powershell
nslookup <destination-fqdn>
```

Expected evidence:

* FQDN resolves successfully
* Resolved IP is expected
* DNS is not pointing to the wrong environment or stale address

### 4. Test basic reachability

Purpose: To check whether the destination network appears reachable.

Commands or checks:

```powershell
ping <destination-ip>
tracert <destination-ip>
```

Expected evidence:

* Ping may succeed if ICMP is allowed
* Traceroute may show where the path stops
* ICMP failure does not always prove the application is unreachable because ICMP may be blocked

### 5. Test required port reachability

Purpose: To test the actual application path instead of relying only on ping.

Commands or checks:

```powershell
Test-NetConnection <destination-ip> -Port <port>
Test-NetConnection <destination-fqdn> -Port <port>
```

Expected evidence:

* If port test succeeds, routing and firewall policy are likely allowing the required traffic
* If port test fails, the issue may be firewall policy, routing, NAT, destination listener, or return path
* If IP works but FQDN fails, DNS, certificate name, proxy, or application virtual host behavior may be involved

### 6. Check routing path

Purpose: To confirm whether traffic has a valid route toward the destination.

Checks:

* Client route table
* Default gateway route
* Router/firewall route table
* Static routes
* Dynamic routing entries
* VPN routes if applicable
* Server-side route back to the client

Expected evidence:

* Source side has a route toward the destination
* Firewall/router has a route toward the destination
* Destination/server side has a route back to the source
* No missing route or wrong next hop is found

### 7. Check firewall/security policy

Purpose: To confirm whether policy allows the required traffic.

Checks:

* Source zone/interface
* Destination zone/interface
* Source IP/subnet
* Destination IP/subnet
* Application or service object
* Protocol and port
* User or identity condition if applicable
* Rule order
* Implicit deny behavior
* Logging setting

Expected evidence:

* A matching allow rule exists
* The allow rule is above any deny rule
* The service/port object is correct
* Logs show allow or deny action
* No implicit deny is blocking the traffic

### 8. Check NAT behavior if applicable

Purpose: To confirm whether NAT is required and correctly applied.

Checks:

* Source NAT rule
* Destination NAT rule
* NAT rule order
* Translated source/destination IP
* Firewall session table
* Return traffic behavior

Expected evidence:

* NAT rule matches the expected traffic
* Translated IP is correct
* Return traffic maps back to the correct session
* No NAT mismatch or missing translation is present

### 9. Check server-side listener and local firewall

Purpose: To avoid blaming the network when the server is not accepting connections.

Checks:

Windows:

```powershell
netstat -ano | findstr :<port>
Get-NetFirewallRule
```

Linux:

```bash
ss -tulpen
sudo iptables -L
sudo firewall-cmd --list-all
```

Expected evidence:

* Server is listening on the expected port
* Server local firewall allows the connection
* Application logs show connection attempts or errors

### 10. Review packet capture or firewall logs

Purpose: To identify where the traffic stops.

Tools or checks:

```text
Firewall traffic logs
Firewall session table
Packet capture on firewall
Packet capture on client
Packet capture on server
Router logs
Server logs
```

Expected evidence:

* Traffic leaves the client
* Traffic reaches the firewall/router
* Policy action is allow or deny
* NAT translation is visible if applicable
* Traffic reaches the server
* Return traffic comes back through the expected path

## Decision Points

| Observation                                         | More Likely Cause                                     |
| --------------------------------------------------- | ----------------------------------------------------- |
| No route to destination                             | Routing issue                                         |
| Route exists but firewall logs show deny            | Firewall policy issue                                 |
| Firewall allows traffic but no server response      | Server listener, local firewall, or return path issue |
| Request reaches server but response does not return | Return path or asymmetric routing issue               |
| NAT translation is wrong                            | NAT rule issue                                        |
| Ping fails but port test succeeds                   | ICMP blocked, application path works                  |
| TCP SYN has no SYN-ACK                              | Firewall, routing, NAT, or server listener issue      |
| TCP connects but application fails                  | Application-layer issue                               |

## Possible Root Causes

| Area               | Possible Cause                             | Evidence                                          |
| ------------------ | ------------------------------------------ | ------------------------------------------------- |
| Routing            | Missing route or wrong next hop            | Route table does not contain valid path           |
| Firewall Policy    | Required traffic blocked                   | Firewall logs show deny or implicit deny          |
| Rule Order         | Correct rule exists but is below deny rule | Earlier rule matches and blocks traffic           |
| Service Object     | Wrong port or protocol configured          | Policy allows incorrect service                   |
| NAT                | Missing or incorrect NAT rule              | Session shows wrong translation or no translation |
| Return Path        | Server cannot route back to client         | Request arrives but response does not return      |
| Asymmetric Routing | Forward and return path differ             | Firewall drops traffic due to state mismatch      |
| Server             | Service not listening                      | Port test fails and server shows no listener      |
| Local Firewall     | Server blocks inbound traffic              | Network path works but host blocks connection     |
| DNS                | FQDN points to wrong IP                    | `nslookup` returns unexpected destination         |

## Verification

The issue is resolved when:

* Source and destination are confirmed
* Required protocol and port are confirmed
* DNS resolves to the correct destination if FQDN is used
* Routing path exists in both directions
* Firewall policy allows the required traffic
* NAT works correctly if required
* Server is listening on the expected port
* Application connection succeeds
* Logs confirm successful traffic flow
* Root cause and verification steps are documented

## Plain-Language Summary

A routing issue means traffic does not know how to reach the destination.

A firewall policy issue means traffic may have a path, but security rules do not allow it to pass.

When troubleshooting, I would not assume the firewall is the problem immediately. I would first confirm the source, destination, protocol, port, routing path, firewall policy, NAT behavior, and return path.

This helps separate a missing path problem from a blocked traffic problem.

## Reflection

One assumption that could be wrong is that every blocked connection is caused by firewall policy. The issue may actually be a missing route, wrong next hop, incorrect NAT, server-side local firewall, application listener issue, or return-path problem.

Another assumption that could be wrong is that successful routing means the application should work. Routing only provides a path. Firewall policy, NAT, server listener, and application behavior still need to be validated.

The most useful evidence would be:

* Source and destination IP
* Required protocol and port
* Route table
* Firewall logs
* Session table
* NAT translation
* Port reachability test
* Server listener status
* Packet capture showing where traffic stops

The troubleshooting result should document:

* Source and destination
* Required service/port
* Expected traffic path
* Routing findings
* Firewall policy findings
* NAT findings
* Server-side findings
* Final root cause area
* Verification result

To make troubleshooting faster next time, I would first define the exact traffic flow, then check routing, firewall policy, NAT, and return path in that order. This prevents me from blaming the firewall too early without proving whether the route and server-side path are correct.
