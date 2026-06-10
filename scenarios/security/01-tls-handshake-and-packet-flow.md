# Scenario: TLS Handshake and Packet Flow

## Scenario Summary

A client is trying to connect securely to a server using HTTPS or another TLS-protected service. The goal is to explain how TLS works, what should appear in packet flow, and how to troubleshoot when the TLS connection fails.

TLS stands for Transport Layer Security. It provides encryption, server identity validation, and integrity protection for communication between a client and a server.

## Initial Framing

Before jumping into TLS details, I would separate the connection into several stages:

* Name resolution
* TCP connectivity
* TLS negotiation
* Certificate validation
* Encrypted application data
* Application response

The key point is that TLS does not happen first. The client usually needs DNS resolution and a successful TCP connection before the TLS handshake can begin.

## Plain-Language Explanation

TLS is like creating a secure private conversation before sending sensitive information.

First, the client and server agree on how they will protect the communication. Then the server proves its identity using a certificate. After that, both sides create shared encryption keys. Once the secure session is ready, the actual application data is sent in encrypted form.

In HTTPS, this means the browser and web server complete the TLS handshake before the HTTP request and response are exchanged securely.

## Expected High-Level Packet Flow

For a typical HTTPS connection, I would expect to see this sequence:

```text
Client
  |
  | DNS query for server name
  |
  | TCP SYN
  | TCP SYN-ACK
  | TCP ACK
  |
  | TLS ClientHello
  | TLS ServerHello
  | Certificate / key exchange / handshake messages
  | TLS Finished
  |
  | Encrypted Application Data
  |
Server
```

A simplified flow is:

```text
DNS resolution
→ TCP three-way handshake
→ TLS handshake
→ encrypted application data
→ application response
```

## Technical Explanation

### 1. DNS resolution

The client resolves the server name to an IP address.

Example:

```powershell
nslookup example.com
```

If DNS fails, the client may never reach the correct destination.

### 2. TCP three-way handshake

Before TLS starts, the client must establish a TCP session with the server.

For HTTPS, this is usually TCP port 443.

Expected packet flow:

```text
Client → Server: SYN
Server → Client: SYN-ACK
Client → Server: ACK
```

If this fails, the issue is not yet TLS. It may be routing, firewall, NAT, server availability, or port reachability.

### 3. TLS ClientHello

After TCP is established, the client sends a TLS ClientHello.

The ClientHello may include:

* Supported TLS versions
* Supported cipher suites
* SNI, which indicates the requested hostname
* ALPN, such as HTTP/2 or HTTP/1.1
* Random value used for key generation

If the server requires SNI and the client does not send the correct hostname, the server may return the wrong certificate or reject the connection.

### 4. TLS ServerHello and certificate exchange

The server responds with TLS negotiation information.

Depending on TLS version, the visible packet details may differ. In general, the server selects supported parameters and provides identity information through a certificate chain.

The certificate should be checked for:

* Correct hostname
* Valid expiration date
* Trusted issuing CA
* Complete certificate chain
* Appropriate key usage

### 5. Key exchange and Finished messages

The client and server complete the handshake and derive shared session keys.

After the handshake completes, the connection switches to encrypted communication.

### 6. Encrypted application data

Once TLS is established, the application data is encrypted.

For HTTPS, the HTTP request and response are protected inside TLS.

At this point, a packet capture usually shows encrypted application data rather than readable HTTP content.

## Investigation Flow

### 1. Confirm DNS resolution

Purpose: To verify that the client resolves the correct server name.

Commands or checks:

```powershell
nslookup <server-fqdn>
```

Expected evidence:

* DNS server responds
* FQDN resolves to the expected IP address
* The result is not stale or pointing to the wrong environment

### 2. Confirm TCP port reachability

Purpose: To confirm that the client can establish a TCP connection before troubleshooting TLS.

Commands or checks:

```powershell
Test-NetConnection <server-fqdn> -Port 443
```

Expected evidence:

* TCP test succeeds if the port is reachable
* If TCP fails, investigate routing, firewall, NAT, or server listener before TLS

### 3. Inspect TLS handshake from the client side

Purpose: To confirm whether the TLS handshake succeeds and what certificate is presented.

Commands or checks:

```bash
openssl s_client -connect <server-fqdn>:443 -servername <server-fqdn>
```

Expected evidence:

* Server presents a certificate
* Certificate chain is visible
* TLS version and cipher are negotiated
* Verification result indicates whether the certificate is trusted

### 4. Check certificate validity

Purpose: To confirm that the server certificate can be trusted by the client.

Checks:

* Certificate subject or SAN contains the expected hostname
* Certificate is not expired
* Certificate chain includes required intermediate certificates
* Client trusts the root CA
* System time on the client is correct

Expected evidence:

* Certificate matches the requested hostname
* Certificate chain validates successfully
* No expiration or trust errors are present

### 5. Check TLS version and cipher compatibility

Purpose: To confirm that the client and server support at least one common TLS version and cipher suite.

Checks:

* Server supports modern TLS versions
* Client is not restricted to outdated TLS versions
* Security policy does not block the required cipher suite
* Load balancer or proxy TLS settings are correct

Expected evidence:

* Client and server negotiate a common TLS version and cipher suite
* No handshake failure occurs due to protocol or cipher mismatch

### 6. Review packet capture if needed

Purpose: To identify where the connection fails in the packet flow.

Tools:

```text
Wireshark
tcpdump
Firewall packet capture
Load balancer logs
Server logs
```

Expected evidence:

* DNS lookup occurs
* TCP three-way handshake completes
* TLS ClientHello is sent
* ServerHello or TLS alert is received
* Encrypted application data appears after successful handshake

If the flow stops before TCP completes, the problem is not TLS yet.

If TCP completes but TLS fails, the issue may be certificate, TLS version, cipher suite, SNI, proxy inspection, or server-side TLS configuration.

## Expected Packet Flow Patterns

| Observation                        | Possible Meaning                                           |
| ---------------------------------- | ---------------------------------------------------------- |
| DNS fails                          | Client cannot resolve server name                          |
| TCP SYN has no SYN-ACK             | Firewall, routing, NAT, or server listener issue           |
| TCP connects but no TLS response   | Server-side TLS service issue or middlebox issue           |
| TLS alert after ClientHello        | Version, cipher, SNI, or policy mismatch                   |
| Certificate warning                | Expired, untrusted, incomplete chain, or hostname mismatch |
| Handshake succeeds but app fails   | Application, authentication, HTTP status, or backend issue |
| Encrypted application data appears | TLS handshake likely completed successfully                |

## Possible Root Causes

| Area           | Possible Cause                        | Evidence                                           |
| -------------- | ------------------------------------- | -------------------------------------------------- |
| DNS            | Wrong destination IP                  | FQDN resolves to unexpected IP                     |
| TCP            | Port 443 blocked or closed            | TCP connection fails                               |
| Certificate    | Expired or hostname mismatch          | Browser or OpenSSL certificate error               |
| Trust Chain    | Missing intermediate CA               | Certificate verification fails                     |
| Client         | Old TLS version or unsupported cipher | Handshake failure or TLS alert                     |
| Server         | Misconfigured TLS settings            | Server logs or failed handshake                    |
| SNI            | Wrong or missing server name          | Wrong certificate or rejected handshake            |
| Proxy/Firewall | TLS inspection or policy block        | Connection fails at middlebox                      |
| Application    | TLS succeeds but app response fails   | Encrypted data exists but application error occurs |

## Verification

The issue is resolved when:

* DNS resolves to the correct IP address
* TCP port 443 is reachable
* TLS handshake completes successfully
* Certificate matches the hostname and is trusted
* Client and server negotiate a supported TLS version and cipher
* Encrypted application data is exchanged
* Application response is successful
* Root cause and verification steps are documented

## Plain-Language Summary

TLS creates a secure connection between a client and a server.

Before TLS starts, the client usually needs to resolve the server name and establish a TCP connection. Then the client and server negotiate security settings, validate the server certificate, create shared encryption keys, and begin sending encrypted application data.

If a secure connection fails, I would not assume the certificate is the only problem. I would first confirm DNS, TCP reachability, the TLS handshake, certificate trust, protocol compatibility, and finally the application response.

## Reflection

One assumption that could be wrong is that every HTTPS failure is a certificate issue. The problem may happen before TLS starts, such as DNS failure, firewall block, routing issue, NAT issue, or closed TCP port.

Another assumption that could be wrong is that a successful TCP connection means the application is working. TCP only confirms transport reachability. TLS and application-layer behavior still need to be validated.

The most useful evidence would be:

* DNS result
* TCP port reachability
* TLS handshake result
* Certificate details
* TLS version and cipher suite
* TLS alert messages
* Packet capture showing where the flow stops
* Server, firewall, proxy, or load balancer logs

The troubleshooting result should document:

* Source client
* Destination FQDN and IP address
* Required port
* DNS result
* TCP reachability result
* TLS handshake result
* Certificate validation result
* TLS version and cipher
* Final root cause area

To make troubleshooting faster next time, I would follow the connection order: DNS first, TCP second, TLS third, and application response last. This prevents me from troubleshooting certificate or application issues before proving that the lower layers are working.

