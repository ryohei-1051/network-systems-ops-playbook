# Scenario: VPN Tunnel Is Up but Traffic Does Not Pass

## Scenario Summary

A VPN tunnel shows as up, but users or servers still cannot reach resources across the VPN.

The goal is to explain why VPN tunnel status alone does not prove application traffic is working, and how to troubleshoot the path from source to destination.

The key lesson is:

> A VPN tunnel being up means the control/security negotiation may be established, but it does not guarantee that routing, firewall policy, NAT, encryption domains, or return traffic are working correctly.

## Initial Framing

Before assuming the VPN itself is broken, I would separate the issue into several areas:

* Tunnel status
* Interesting traffic / encryption domains / proxy IDs
* Routing
* Firewall policy
* NAT exemption
* Return path
* Packet capture or VPN logs
* Application/service reachability

The important point is to confirm both the tunnel control plane and the actual data traffic flow.

## Plain-Language Explanation

A VPN tunnel being up is like having a secure road built between two sites.

However, even if the road exists, traffic still needs:

* A correct route to enter the road
* Permission to pass through security gates
* The correct destination address
* A valid return path
* No accidental address translation that breaks the flow

So when a VPN tunnel is up but traffic fails, I would not stop at tunnel status. I would verify whether the actual application traffic is entering the tunnel, being allowed, reaching the remote side, and returning successfully.

## Expected Behavior

A working VPN connection should allow traffic to:

* Match the correct VPN tunnel or encryption domain
* Route toward the VPN
* Pass local firewall policy
* Avoid incorrect NAT if private subnets should stay unchanged
* Cross the VPN tunnel
* Pass remote firewall policy
* Reach the destination server and port
* Return through the correct path
* Complete the application connection

## Clarifying Questions

* Is this site-to-site VPN or remote-access VPN?
* Is the tunnel showing up on both sides?
* Is the issue affecting all traffic or only one subnet, host, or application?
* What are the source and destination IP addresses?
* What protocol and port should work?
* Did this work before?
* Were there recent changes to routing, firewall policy, NAT, VPN selectors, or remote networks?
* Is NAT required or should NAT be exempted?
* Are both sides using matching encryption domains or proxy IDs?
* Are logs or packet captures available from both VPN gateways?

## Investigation Flow

### 1. Confirm tunnel status

Purpose: To verify whether the VPN tunnel is established.

Checks:

```text
VPN tunnel status
IKE/Phase 1 status
IPsec/Phase 2 status
Tunnel uptime
Remote peer status
VPN gateway logs
```

Expected evidence:

* Tunnel is established on both sides
* No repeated negotiation failure
* Phase 1 and Phase 2 are stable
* No authentication, proposal, or peer mismatch errors

Important note:

Tunnel status alone does not prove that application traffic is passing.

### 2. Confirm source, destination, protocol, and port

Purpose: To define the exact traffic flow before troubleshooting.

Information to collect:

```text
Source IP:
Source subnet:
Destination IP:
Destination subnet:
Protocol:
Port:
Expected application:
Local VPN gateway:
Remote VPN gateway:
```

Expected evidence:

* Traffic flow is clearly defined
* Required service or application port is known
* Source and destination are part of the expected VPN policy

### 3. Confirm encryption domain / proxy ID / traffic selector

Purpose: To verify that the traffic is allowed to enter the VPN tunnel.

Checks:

* Local subnet
* Remote subnet
* Proxy ID / traffic selector
* Phase 2 selector
* Interesting traffic ACL
* Route-based or policy-based VPN behavior

Expected evidence:

* Source subnet matches the local encryption domain
* Destination subnet matches the remote encryption domain
* Both VPN peers have matching selectors
* Traffic is not excluded because of a subnet mismatch

### 4. Confirm routing toward the VPN

Purpose: To verify that traffic is being sent toward the VPN gateway.

Checks:

Windows:

```powershell
route print
tracert <remote-destination-ip>
```

Linux:

```bash
ip route
traceroute <remote-destination-ip>
```

Firewall/router:

```text
Routing table
Static routes
Dynamic routes
VPN route table
Policy-based forwarding if used
```

Expected evidence:

* Source network has a route to the remote VPN subnet
* Firewall or router sends traffic toward the VPN tunnel
* No default route or more specific route sends traffic the wrong way

### 5. Check firewall policy

Purpose: To confirm that local and remote security policies allow the VPN traffic.

Checks:

* Local firewall policy from source zone to VPN zone/tunnel
* Remote firewall policy from VPN zone/tunnel to destination zone
* Required protocol and port
* Rule order
* Implicit deny
* Logging
* User or identity conditions if applicable

Expected evidence:

* Matching allow policy exists
* Policy is above any deny rule
* Required service object is correct
* Logs show traffic allowed, not denied

### 6. Check NAT exemption or NAT behavior

Purpose: To confirm whether NAT is interfering with the VPN traffic.

Checks:

* Source NAT rules
* Destination NAT rules
* NAT exemption / no-NAT rules
* NAT rule order
* Translated source and destination IPs
* Session table

Expected evidence:

* Internal source IP is preserved if NAT exemption is required
* NAT rules do not accidentally translate VPN traffic
* Remote side receives the expected source IP
* Return traffic matches the correct session

### 7. Test required port reachability

Purpose: To test the actual application path.

Commands or checks:

```powershell
Test-NetConnection <remote-destination-ip> -Port <port>
```

Linux:

```bash
nc -vz <remote-destination-ip> <port>
```

Expected evidence:

* If port test succeeds, VPN path, routing, policy, and service are likely working
* If port test fails, investigate policy, routing, NAT, server listener, or return path
* Ping failure alone is not conclusive because ICMP may be blocked

### 8. Check destination server and local firewall

Purpose: To avoid blaming the VPN when the destination server is not accepting traffic.

Checks:

Windows:

```powershell
netstat -ano | findstr :<port>
Get-NetFirewallRule
```

Linux:

```bash
ss -tulpen
sudo firewall-cmd --list-all
sudo iptables -L
```

Expected evidence:

* Server is listening on the expected port
* Server local firewall allows traffic from the VPN source subnet
* Application logs show connection attempts or errors

### 9. Check return path

Purpose: To confirm that the destination can send traffic back to the source.

Checks:

* Destination server default gateway
* Remote firewall route back to source subnet
* Remote-side VPN route
* Remote firewall policy
* NAT behavior on return traffic
* Asymmetric routing possibility

Expected evidence:

* Server has a valid return path to the source subnet
* Return traffic enters the VPN tunnel
* No asymmetric path breaks stateful firewall inspection
* Firewall/session logs show bidirectional traffic

### 10. Review logs and packet capture

Purpose: To identify exactly where the traffic stops.

Tools or checks:

```text
VPN gateway logs
Firewall traffic logs
Session table
Packet capture on local VPN gateway
Packet capture on remote VPN gateway
Packet capture on destination server
Application logs
```

Expected evidence:

* Traffic leaves the source
* Traffic enters the local VPN gateway
* Traffic matches the VPN tunnel
* Traffic exits the remote VPN gateway
* Traffic reaches the destination server
* Return traffic comes back through the VPN

## Decision Points

| Observation                                    | More Likely Cause                                                      |
| ---------------------------------------------- | ---------------------------------------------------------------------- |
| Tunnel is down                                 | VPN negotiation, peer, proposal, authentication, or connectivity issue |
| Tunnel is up but no traffic enters tunnel      | Routing, selector, or policy issue                                     |
| Traffic matches wrong tunnel or no selector    | Encryption domain / proxy ID mismatch                                  |
| Traffic is denied in logs                      | Firewall policy or rule order issue                                    |
| Traffic is NATed unexpectedly                  | NAT exemption or NAT rule order issue                                  |
| Traffic reaches remote gateway but not server  | Remote firewall, routing, or destination issue                         |
| Traffic reaches server but no response returns | Return path, local firewall, or asymmetric routing issue               |
| Ping fails but port test works                 | ICMP blocked but application path works                                |
| TCP SYN leaves but no SYN-ACK returns          | Remote firewall, server listener, return path, or NAT issue            |

## Possible Root Causes

| Area               | Possible Cause                        | Evidence                                         |
| ------------------ | ------------------------------------- | ------------------------------------------------ |
| VPN Tunnel         | Phase 1 or Phase 2 unstable           | Tunnel flaps or negotiation errors               |
| Selectors          | Encryption domain/proxy ID mismatch   | Tunnel up but traffic does not match             |
| Routing            | Missing route to remote subnet        | Traffic does not enter VPN path                  |
| Firewall Policy    | VPN traffic blocked                   | Logs show deny or implicit deny                  |
| NAT                | VPN traffic is translated incorrectly | Remote side sees unexpected source IP            |
| NAT Exemption      | No-NAT rule missing                   | Private subnet traffic is NATed before tunnel    |
| Return Path        | Remote side cannot route back         | Request arrives but response does not return     |
| Asymmetric Routing | Return traffic uses different path    | Stateful firewall drops return traffic           |
| Server             | Service not listening                 | Port test fails and no listener is present       |
| Local Firewall     | Destination host blocks traffic       | Packet reaches host but host rejects or drops it |

## Verification

The issue is resolved when:

* VPN tunnel is stable
* Source and destination subnets match the expected VPN selectors
* Route points traffic toward the VPN
* Firewall policy allows the required traffic on both sides
* NAT exemption or NAT behavior is correct
* Destination server is listening on the expected port
* Return path is confirmed
* Application traffic succeeds
* Logs or packet captures confirm bidirectional flow
* Root cause and verification steps are documented

## Plain-Language Summary

A VPN tunnel being up does not always mean that application traffic is working.

The tunnel status only confirms that the secure connection between VPN gateways may be established. The actual traffic still needs correct routing, firewall rules, NAT behavior, matching VPN selectors, and a valid return path.

When troubleshooting, I would first confirm the tunnel status, then check whether the specific source-to-destination traffic is entering the tunnel, allowed by policy, not broken by NAT, reaching the server, and returning successfully.

## Reflection

One assumption that could be wrong is that a VPN tunnel showing “up” means the network path is fully working. In reality, the tunnel may be up while application traffic fails because of routing, firewall policy, NAT, selector mismatch, server listener, or return-path problems.

Another assumption that could be wrong is that ping failure means VPN failure. ICMP may be blocked while the required application port is still working.

The most useful evidence would be:

* Tunnel status
* Source and destination IP/subnet
* VPN selectors or proxy IDs
* Routing table
* Firewall traffic logs
* NAT/session table
* Packet capture on both VPN gateways
* Destination server listener status
* Return path verification

The troubleshooting result should document:

* VPN type
* Local and remote peer
* Source and destination subnets
* Required protocol and port
* Tunnel status
* Selector/proxy ID match
* Routing findings
* Firewall policy findings
* NAT findings
* Server-side findings
* Final root cause area
* Verification result

To make troubleshooting faster next time, I would avoid stopping at tunnel status. I would verify the full traffic path: source, route, VPN selector, policy, NAT, remote gateway, server, and return path.
