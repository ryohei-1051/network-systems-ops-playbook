# Network & Systems Operations Playbook

## Overview

This repository is a scenario-based operations playbook focused on structured troubleshooting, technical explanation, and infrastructure reasoning.

The goal is to document how I approach network and systems problems in a clear, evidence-driven way before jumping directly into commands or a single hypothesis.

This project reflects practical operations skills across:

- Network troubleshooting
- Systems administration
- Security and packet-flow analysis
- Plain-language technical communication
- Handling unfamiliar infrastructure safely
- Post-scenario reflection and improvement

## Why This Project Exists

In real infrastructure operations, technical knowledge is only one part of the job.

A strong engineer also needs to:

- Frame the problem clearly
- Confirm scope and impact
- Separate symptoms from root cause
- Build and test hypotheses
- Explain technical issues to both technical and non-technical audiences
- Document findings in a reusable format
- Reflect on what could be improved next time

This repository is designed to strengthen those habits through repeated scenario-based documentation.

## Repository Structure

```text
docs/            Reusable frameworks and methodology notes
scenarios/       Scenario-based troubleshooting and explanation files
templates/       Reusable markdown templates
retrospectives/  Weekly review notes and improvement tracking
assets/          Diagrams and supporting visuals
```

## Main Frameworks Used

This project uses four reusable answer frameworks:
| Area                               | Framework | Purpose                                           |
| ---------------------------------- | --------- | ------------------------------------------------- |
| Troubleshooting                    | SCOPE     | Structure technical investigation before commands |
| Unfamiliar Technology              | RAMP      | Safely approach unknown devices or platforms      |
| Plain-Language Explanation         | LAYER     | Explain technical concepts clearly                |
| Behavioral / Operations Reflection | STAR-R    | Connect actions to results and lessons learned    |

## Example Scenario Areas

- Home internet troubleshooting
- DNS resolution succeeds but server access fails
- DHCP DORA and client IP behavior
- TTL behavior and troubleshooting value
- TLS handshake and packet-flow expectations
- Operating an unfamiliar network switch
- MPLS explained in plain language
- VPN tunnel is up but traffic does not pass
- Firewall policy versus routing issue
- Service is running but application is unavailable

## Quality Standard

Each scenario should show:
- Initial problem framing
- Assumptions and constraints
- Step-by-step reasoning
- Evidence to collect
- Commands or checks only after the problem is framed
- Likely root causes
- Verification steps
- Plain-language summary
- Reflection for improvement

## Portfolio Value

This project demonstrates practical infrastructure thinking beyond isolated commands.
It shows how I communicate, troubleshoot, document, and improve operational decision-making across network and systems environments.

## Status
4-week structured project in progress.

## Scenario Index

### Network

- [Home Internet Troubleshooting](scenarios/network/01-home-internet-troubleshooting.md)
- [DNS Resolves but Server Is Unreachable](scenarios/network/02-client-can-resolve-dns-but-cannot-reach-server.md)
- [TTL Explained and Used for Troubleshooting](scenarios/network/03-ttl-explained-and-troubleshot.md)
- [DHCP DORA and Client IP Acquisition](scenarios/network/04-dhcp-dora-client-ip-acquisition.md)
- [Operating an Unfamiliar Switch](scenarios/network/05-unfamiliar-switch-operation.md)