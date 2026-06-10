# Week 02 Security and Packet Flow Review

## Review Scope

This review covers the first set of security and packet-flow scenarios in the operations playbook:

* TLS Handshake and Packet Flow
* Firewall Policy Versus Routing Issue
* VPN Tunnel Is Up but Traffic Does Not Pass

## Purpose of This Review

The purpose of this review is to evaluate whether the security scenarios are improving structured troubleshooting and packet-flow reasoning.

The main focus areas are:

* Following the connection sequence in the correct order
* Separating lower-layer reachability from application-layer behavior
* Avoiding assumptions based on one successful checkpoint
* Identifying where traffic stops
* Using logs, packet captures, and verification evidence

## Main Pattern Learned

The biggest pattern across these scenarios is that a successful status check does not always prove that the full application path is working.

Examples:

* DNS success does not prove server reachability
* TCP success does not prove TLS success
* TLS success does not prove application success
* VPN tunnel status does not prove application traffic is passing
* A route existing does not prove firewall policy allows the traffic
* Firewall policy allowing traffic does not prove the server is listening
* A request reaching the server does not prove the return path is working

The better approach is to verify each stage of the flow in order.

## Strongest Security Scenario So Far

The strongest security scenario so far is:

**VPN Tunnel Is Up but Traffic Does Not Pass**

This scenario is strong because it separates tunnel status from actual data-plane traffic.

The key lesson is:

> A VPN tunnel being up means the tunnel negotiation may be established, but it does not guarantee that routing, firewall policy, NAT, selectors, server listener, or return traffic are working correctly.

This is useful because it prevents stopping too early at a surface-level status check.

## Best Packet-Flow Scenario So Far

The best packet-flow scenario so far is:

**TLS Handshake and Packet Flow**

This scenario is useful because it shows the correct order of a secure connection:

1. DNS resolution
2. TCP three-way handshake
3. TLS ClientHello and ServerHello
4. Certificate validation
5. Key exchange and Finished messages
6. Encrypted application data
7. Application response

The key lesson is:

> TLS troubleshooting should not start with certificates until DNS and TCP reachability are confirmed.

## Most Practical Troubleshooting Scenario

The most practical troubleshooting scenario so far is:

**Firewall Policy Versus Routing Issue**

This scenario is useful because it separates different failure types:

* Routing issue: traffic does not have a valid path
* Firewall policy issue: traffic has a path but is not allowed
* NAT issue: traffic is translated incorrectly
* Return-path issue: request may arrive, but response cannot get back
* Server-side issue: network path may work, but the service is not listening

The key lesson is:

> Not every connection failure is a firewall issue. The route, NAT behavior, server listener, and return path must also be validated.

## Before and After Answer Pattern

Before this project, an answer might start too quickly with one possible cause.

Example:

> I would check the firewall rule and see if the traffic is blocked.

A stronger answer starts by defining the flow.

Example:

> Before assuming it is a firewall issue, I would define the source, destination, protocol, port, expected path, NAT requirement, and return path. Then I would check routing first, firewall policy second, NAT behavior third, and server-side listener or logs after that.

The second answer is stronger because it shows a repeatable troubleshooting method rather than a single hypothesis.

## Senior-Quality Habits Reinforced

This week reinforced several senior-quality habits:

* Do not stop at surface-level status
* Verify the actual traffic flow
* Separate control-plane status from data-plane behavior
* Check both forward path and return path
* Confirm the required protocol and port
* Use logs and packet captures to locate where traffic stops
* Avoid blaming the firewall before checking routing, NAT, and server-side behavior
* Document evidence, not just conclusions

## Common Assumptions to Avoid

These assumptions can lead to weak troubleshooting:

| Assumption                                   | Why It May Be Wrong                                                    |
| -------------------------------------------- | ---------------------------------------------------------------------- |
| DNS works, so the connection should work     | DNS only confirms name-to-IP resolution                                |
| TCP connects, so TLS should work             | TLS can still fail due to certificate, version, cipher, or SNI issues  |
| The VPN is up, so traffic should pass        | Routing, selectors, NAT, policy, or return path can still fail         |
| Ping fails, so the service is down           | ICMP may be blocked while the application port works                   |
| Firewall is the problem                      | Routing, NAT, server listener, or local firewall may be the real issue |
| Request reaches destination, so path is good | Return traffic may fail or use an asymmetric path                      |

## Evidence Checklist

For future security and packet-flow scenarios, I should consistently collect:

* Source IP
* Destination IP or FQDN
* Protocol and port
* DNS result
* Route table
* Firewall policy result
* NAT/session table
* Port reachability test
* Server listener status
* Application log result
* Packet capture showing where traffic stops
* Return-path confirmation

## What to Improve Next

For the next set of scenarios, I should improve three areas:

### 1. Shorter first answer

Some scenario files are intentionally detailed, but in a real conversation, the first answer should be concise.

A good first answer should be:

* Frame the issue
* Identify the main checkpoints
* Mention the safest first evidence to collect
* Expand only if asked

### 2. More decision points

Future scenarios should include more conditional reasoning:

* If this succeeds, what does it prove?
* If this fails, what does it suggest?
* What does it not prove?
* What would I check next?

### 3. More production-awareness

Future scenarios should include:

* Change windows
* Rollback
* Logging
* Impact scope
* Escalation points
* Documentation updates

## Next Scenario Candidates

Good next scenarios include:

* Service is running but application is unavailable
* User can log in but cannot access a share
* MPLS explained in plain language
* Certificate warning after internal service migration
* Slow application but ping is normal

## Final Reflection

The security and packet-flow scenarios helped reinforce one major habit:

> Troubleshooting should follow the actual connection flow, not the first technology name mentioned in the problem.

If the issue is HTTPS, I should still verify DNS and TCP before TLS.

If the issue is VPN, I should still verify routing, policy, NAT, and return path before concluding the VPN itself is broken.

If the issue sounds like a firewall problem, I should still prove whether traffic has a route, matches the correct policy, reaches the server, and returns successfully.

This is the operational thinking pattern I want to continue building toward senior-level systems, network, and security troubleshooting.
