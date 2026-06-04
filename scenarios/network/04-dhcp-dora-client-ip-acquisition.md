# Scenario: DHCP DORA and Client IP Acquisition

## Scenario Summary

A client needs to obtain an IP address automatically from a DHCP server. The goal is to explain how DHCP DORA works and how to troubleshoot when the client does not receive a valid IP address.

DHCP DORA stands for:

- Discover
- Offer
- Request
- Acknowledge

This process allows a client to receive network configuration such as an IP address, subnet mask, default gateway, DNS server, and lease time.

## Initial Framing

Before jumping into commands, I would separate the issue into two parts:

- How the DHCP process should work
- Where the process may be failing

The key point is that DHCP is not only about receiving an IP address. It also provides other important network settings that the client needs to communicate properly.

## Plain-Language Explanation

DHCP is like checking in at a hotel.

The client first asks, “Is there any available room?”  
The DHCP server replies, “Yes, you can use this room.”  
The client then says, “I want to use that room.”  
The server confirms, “Approved. You can use it for this period of time.”

In networking terms, the “room” is the IP address lease.

## Technical Explanation

The DHCP process normally follows four steps:

### 1. Discover

The client broadcasts a DHCP Discover message because it does not yet know which DHCP server is available.

### 2. Offer

A DHCP server responds with an available IP address and network configuration.

### 3. Request

The client requests to use the offered IP address.

### 4. Acknowledge

The DHCP server confirms the lease and the client applies the configuration.

After this process, the client should have:

- IP address
- Subnet mask
- Default gateway
- DNS server
- Lease information

## Client Behavior

If DHCP succeeds, the client configures the leased IP address and uses the assigned gateway and DNS servers.

If DHCP fails, the client may assign itself an APIPA address such as `169.254.x.x` on Windows. This usually means the client could not receive a valid DHCP lease.

If the client has an APIPA address, it may communicate only with other devices on the same local link that also use APIPA, but it usually cannot reach the default gateway or the Internet.

## Clarifying Questions

- Is the issue affecting one client or multiple clients?
- Is the client using DHCP or static IP configuration?
- Is the client connected by wired Ethernet or Wi-Fi?
- Did the issue start after a network, VLAN, DHCP scope, or switch change?
- Does the client receive an APIPA address?
- Are other clients in the same VLAN receiving DHCP addresses?
- Is the DHCP server local or reached through a DHCP relay/helper address?

## Expected Behavior

The client should be able to:

- Send DHCP Discover
- Receive DHCP Offer
- Send DHCP Request
- Receive DHCP Acknowledge
- Apply a valid IP address
- Use the assigned default gateway and DNS servers

## Investigation Flow

### 1. Confirm client IP configuration

Purpose: To check whether the client has a valid DHCP lease.

Commands or checks:

```powershell
ipconfig /all
```

Expected evidence:

- DHCP is enabled
- Client has a valid IP address
- Client is not using APIPA
- Default gateway is present
- DNS servers are present
- DHCP server and lease time are visible

### 2. Renew the DHCP lease

Purpose: To force the client to request a new DHCP lease.

Commands or checks:
```powershell
ipconfig /release
ipconfig /renew
```

Expected evidence:
- Client receives a new valid lease
- Error message appears if DHCP fails
- Lease information updates

### 3. Check local connectivity

Purpose: To confirm whether the client can communicate on the local network.

Commands or checks:
```powershell
ping <default-gateway>
```

Expected evidence:
- If the gateway replies, local Layer 3 connectivity is working
- If the gateway does not reply, check adapter, VLAN, switch port, Wi-Fi, or gateway status

### 4. Check DHCP scope and server side

Purpose: To confirm whether the DHCP server can provide valid leases.

Checks:
- DHCP scope is active
- Address pool is not exhausted
- Correct subnet is configured
- Correct default gateway option is configured
- Correct DNS server option is configured
- DHCP service is running

Expected evidence:
- Available addresses exist in the scope
- Client lease appears on the DHCP server
- DHCP server logs show Discover, Offer, Request, and Acknowledge behavior

### 5. Check VLAN, relay, and network path

Purpose: To confirm whether the DHCP request can reach the DHCP server.

Checks:
- Client is in the correct VLAN
- Switch port VLAN assignment is correct
- Trunk allows the VLAN if applicable
- DHCP relay/helper address is configured if the DHCP server is on another subnet
- Firewall is not blocking DHCP traffic

Expected evidence:
- DHCP broadcast reaches the correct server or relay
- Relay forwards the request to the DHCP server
- DHCP response returns to the client

### 6. Check packet-level evidence if needed

Purpose: To identify where the DHCP process stops.

Tools or checks:
```
Wireshark
Packet capture on firewall/switch/server
DHCP server logs
```

Expected evidence:
- Discover only: client is asking but no server response
- Discover and Offer only: server responds but client may not accept or receive properly
- Discover, Offer, Request only: server may not acknowledge
- Full DORA visible: DHCP succeeded, so check another issue such as gateway, DNS, or firewall

## Possible Root Causes
| Area         | Possible Cause                 | Evidence                                         |
| ------------ | ------------------------------ | ------------------------------------------------ |
| Client       | DHCP disabled or adapter issue | DHCP not enabled, no valid lease                 |
| Client       | APIPA address assigned         | Client has `169.254.x.x` address                 |
| DHCP Server  | DHCP service down              | No leases issued, service stopped                |
| DHCP Scope   | Address pool exhausted         | No available leases                              |
| DHCP Options | Missing gateway or DNS option  | Client gets IP but cannot route or resolve names |
| VLAN         | Client in wrong VLAN           | Client receives wrong subnet or no lease         |
| Relay        | Missing DHCP helper address    | DHCP works locally but not across subnet         |
| Firewall     | DHCP traffic blocked           | Discover or Offer missing in packet capture      |

## Verification

The issue is resolved when:
- The client receives a valid DHCP lease
- The client has the correct IP address, subnet mask, gateway, and DNS servers
- The client can reach the default gateway
- The client can resolve DNS names
- The client can reach required internal or Internet resources
- The root cause is documented

## Plain-Language Summary

DHCP automatically gives a device the network information it needs. The client first asks for an address, the server offers one, the client requests it, and the server confirms it. This is the DORA process. If DHCP fails, the client may give itself a temporary address such as 169.254.x.x, but that usually does not allow normal network or Internet access. In troubleshooting, I would check whether the client received a valid lease, whether the DHCP server has available addresses, and whether the network path between the client and DHCP server is working.

## Reflection

One assumption that could be wrong is that DHCP failure always means the DHCP server is down. The problem could also be caused by the client adapter, wrong VLAN, exhausted scope, missing relay/helper address, firewall filtering, or incorrect DHCP options.

The most useful evidence would be:
- Client IP configuration
- Whether APIPA is assigned
- DHCP server lease records
- DHCP scope availability
- VLAN assignment
- DHCP relay/helper configuration
- Packet capture showing where DORA stops

The troubleshooting result should document:
- Client device
- VLAN/subnet
- Assigned IP address or APIPA address
- DHCP server
- DHCP scope status
- Gateway and DNS options
- Whether DHCP relay was involved
- Final root cause area

To make troubleshooting faster next time, I would first determine whether the problem affects one client or many clients, then check whether the client received any lease, and finally identify where the DORA process stops.
