# Scenario: Operating an Unfamiliar Switch

## Scenario Summary

You need to operate or investigate a network switch that uses an unfamiliar vendor, model, or command syntax.

The goal is to approach the device safely, understand its role, collect read-only information first, and avoid making risky changes before confirming the current state.

## Initial Framing

Before touching the configuration, I would separate the situation into four areas:

- Device role and business impact
- Access method and permission level
- Current network state
- Safe change and rollback process

The key point is that an unfamiliar switch is still built around common networking concepts such as interfaces, VLANs, trunks, STP, LACP, routing, ACLs, logs, and configuration save behavior.

## Plain-Language Explanation

Working on an unfamiliar switch is like working in a building you have never entered before.

Before changing anything, I would first understand what the building is used for, which rooms are critical, where the exits are, and how to undo a mistake.

In networking terms, that means understanding the switch role, checking the current configuration, identifying connected devices, and preparing a rollback plan before making changes.

## RAMP Framework

I would use the RAMP approach:

- Role and risk
- Access and current state
- Map to known concepts
- Proceed safely

## Investigation Flow

### 1. Confirm role and risk

Purpose: To understand the switch's function before making assumptions.

Questions or checks:

- Is this an access, distribution, core, management, or lab switch?
- Is it in production?
- What users, servers, APs, phones, or uplinks depend on it?
- Is there redundancy?
- Is there an approved change window if configuration changes are needed?

Expected evidence:

- Device role is understood
- Business or service impact is identified
- Risk level is clear before making changes

### 2. Confirm access and backup

Purpose: To make sure I can safely view the device and recover if needed.

Checks:

- Confirm login method
- Confirm privilege level
- Confirm out-of-band or console access if available
- Export or save the current configuration
- Confirm how configuration is saved or rolled back on that platform

Expected evidence:

- Read-only access is available
- Current configuration is backed up
- Rollback method is understood
- No configuration change has been made yet

### 3. Collect read-only current state

Purpose: To understand the device before changing anything.

Commands or checks may vary by vendor, but I would look for:

```text
show version
show running-config
show interfaces status
show vlan
show interfaces trunk
show spanning-tree
show etherchannel summary
show lldp neighbors
show cdp neighbors
show ip interface brief
show ip route
show logging
```
Expected evidence:

- Hostname, model, version, and uptime
- Interface status
- VLANs and trunk ports
- STP state
- LACP/EtherChannel status
- Neighbor devices
- Routing information if Layer 3 switching is used
- Recent logs or errors

4. Map unfamiliar syntax to known concepts

Purpose: To avoid being blocked by vendor-specific command differences.

I would map the platform to common concepts:
| Concept       | What I Need to Identify                      |
| ------------- | -------------------------------------------- |
| Interfaces    | Port status, speed, duplex, errors           |
| VLANs         | Access VLANs and allowed VLANs               |
| Trunks        | Tagged/untagged VLAN behavior                |
| STP           | Root bridge, blocked ports, topology changes |
| LACP          | Port-channel members and status              |
| Routing       | SVIs, routes, default route                  |
| ACLs          | Traffic filtering or management access       |
| Logs          | Errors, link flaps, authentication failures  |
| Save behavior | Running vs startup config or commit model    |

Expected evidence:
- Vendor-specific syntax is translated into familiar networking behavior
- I understand what each configuration section is doing before changing it

5. Troubleshoot the reported issue

Purpose: To investigate based on symptoms instead of randomly checking commands.

Examples:
- If one user is affected, check access port, VLAN, cable, MAC address table, and port security
- If one VLAN is affected, check SVI, trunk allowed VLANs, STP, DHCP relay, and routing
- If uplink is affected, check interface status, LACP, trunking, errors, and neighbor device
- If management access is affected, check management VLAN, default gateway, ACLs, and AAA settings

Expected evidence:
- Investigation follows the symptom scope
- Evidence is collected before a change is proposed
- The issue is narrowed to client, port, VLAN, uplink, routing, or policy area

6. Proceed with changes safely if needed

Purpose: To minimize risk during configuration changes.

Safe change process:
- Confirm the exact change needed
- Review vendor documentation
- Prepare rollback commands
- Make one change at a time
- Validate immediately after the change
- Save configuration only after validation
- Document what changed and why

Expected evidence:
- Change is controlled and reversible
- Validation confirms the intended result
- No unrelated configuration is modified
- Possible Root Causes

## Possible Root Causes
| Area        | Possible Issue                         | Evidence                                                   |
| ----------- | -------------------------------------- | ---------------------------------------------------------- |
| Access Port | Wrong VLAN or disabled port            | One device affected, port status or VLAN mismatch          |
| Trunk       | VLAN not allowed on trunk              | Entire VLAN affected across uplink                         |
| STP         | Port blocked or topology issue         | STP output shows blocked port or topology changes          |
| LACP        | Port-channel member down               | EtherChannel/LACP summary shows suspended or inactive link |
| Physical    | Cable, SFP, speed, or duplex issue     | Interface errors, down/down state, CRC errors              |
| Routing     | Missing SVI, route, or default gateway | Layer 3 reachability fails                                 |
| Security    | Port security, ACL, or AAA issue       | Logs show blocked access or authentication failure         |
| Management  | Wrong management VLAN or gateway       | Device is forwarding traffic but not manageable            |

## Verification

The issue is resolved or safely understood when:
- Switch role and impact are identified
- Current configuration is backed up
- Relevant interfaces, VLANs, trunks, STP, LACP, routing, and logs are reviewed
- The issue is narrowed based on evidence
- Any change is validated and documented
- Rollback is available if needed

## Plain-Language Summary
If I need to work on an unfamiliar switch, I would not start by guessing commands or changing configuration. First, I would identify what the switch does, how important it is, what devices depend on it, and whether I have a safe way to recover if something goes wrong. Then I would collect read-only information, map the unfamiliar vendor syntax to common switching concepts, and only make a controlled change after I understand the risk and rollback method.

## Reflection
One assumption that could be wrong is that unfamiliar technology requires completely new knowledge. In many cases, the commands are different, but the underlying concepts are familiar: interfaces, VLANs, trunks, STP, LACP, routing, ACLs, and logs.

- The most useful evidence would be:
- Switch role
- Interface status
- VLAN and trunk configuration
- STP state
- LACP status
- Neighbor information
- Logs
- Backup or rollback method

The troubleshooting result should document:
- Device role
- Business impact
- Access method
- Current configuration backup
- Symptom scope
- Commands or checks used
- Findings
- Change made, if any
- Verification result
- Rollback plan

To make troubleshooting faster next time, I would keep a vendor-neutral checklist for switch investigation and add vendor-specific command equivalents as I learn each platform.
