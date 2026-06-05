# Week 01 Scenario Review

## Review Scope

This review covers the first set of network-focused scenarios in the operations playbook:

* Home Internet Troubleshooting
* DNS Resolves but Server Is Unreachable
* TTL Explained and Used for Troubleshooting
* DHCP DORA and Client IP Acquisition
* Operating an Unfamiliar Switch

## Purpose of This Review

The purpose of this review is to identify whether the scenarios are improving structured technical reasoning, not just collecting commands or definitions.

The main focus areas are:

* Framing the problem before troubleshooting
* Separating symptoms from root cause
* Explaining technical concepts in plain language
* Using evidence before choosing a hypothesis
* Showing safe operational behavior

## What Improved

The biggest improvement in this first set of scenarios is that each scenario starts with problem framing before commands.

Instead of immediately jumping to a command or one possible cause, the scenarios now follow a more consistent pattern:

1. Define the issue
2. Clarify scope and expected behavior
3. Identify possible affected layers
4. Collect evidence
5. Test likely causes
6. Verify the result
7. Reflect on what could be improved

This structure makes the answers more controlled and easier to follow.

## Strongest Scenario So Far

The strongest scenario so far is:

**Operating an Unfamiliar Switch**

This scenario is strong because it shows production-safe thinking. The focus is not only on technical commands, but also on role, risk, backup, rollback, read-only investigation, and vendor-neutral concepts.

The key lesson is:

> An unfamiliar device should not be treated as a command memorization problem. It should be treated as an operational risk and discovery problem first.

## Most Practical Troubleshooting Scenario

The most practical troubleshooting scenario so far is:

**DNS Resolves but Server Is Unreachable**

This scenario is useful because it clearly separates DNS resolution from full application reachability.

The key lesson is:

> DNS success only confirms name-to-IP translation. It does not prove that routing, firewall policy, port reachability, or the destination service is working.

## Best Plain-Language Explanation

The best plain-language explanation so far is:

**DHCP DORA and Client IP Acquisition**

The hotel analogy makes the DORA process easier to understand while still connecting the explanation to troubleshooting evidence such as APIPA, DHCP scope, VLAN assignment, relay/helper configuration, and packet capture.

## Weaknesses Noticed

One weakness is that some explanations can still become long. In a real conversation, the first answer should be shorter, then expanded only if needed.

Another weakness is that some scenarios rely on generic commands. To improve quality, future scenarios should include more decision points, such as:

* If this test succeeds, what does it prove?
* If this test fails, what does it rule out?
* What evidence would change the conclusion?
* What is the safest next step?

## Before and After Answer Pattern

Before this project, the answer might start too quickly with a command.

Example:

> I would run ipconfig, ping the gateway, and check DNS.

A stronger answer starts with framing.

Example:

> Before running commands, I would first confirm the scope and expected behavior. I would check whether the issue affects one client, one subnet, or multiple users, then separate the problem into client configuration, local network path, DNS, routing, firewall policy, and destination service availability.

This second version sounds more organized and easier to trust.

## Senior-Quality Habits to Keep Building

The habits to keep building are:

* Confirm scope before troubleshooting
* Identify expected behavior before testing
* Use evidence to narrow hypotheses
* Avoid assuming one root cause too early
* Consider rollback before making changes
* Explain the issue clearly to both technical and non-technical audiences
* Document the final root cause and verification method

## What to Improve in the Next Set

For the next scenarios, I should focus more on packet flow and security-related reasoning.

Good next topics include:

* TLS handshake and packet flow
* Firewall policy versus routing issue
* VPN tunnel is up but traffic does not pass
* Service is running but application is unavailable
* MPLS explained in plain language

## Final Reflection

The first set of scenarios shows progress toward more structured operational thinking.

The main value of this playbook is not the number of scenarios. The value is building a repeatable communication pattern:

> Frame the problem, identify the expected behavior, collect evidence, test hypotheses, verify the result, and document the lesson.

This is the standard I want to keep improving as I build toward senior-level systems, network, and security operations quality.

