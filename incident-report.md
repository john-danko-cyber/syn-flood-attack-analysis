# Cybersecurity Incident Report: SYN Flood Attack

This report covers a guided lab from the Google Cybersecurity Certificate. I reviewed the scenario and the Wireshark TCP/HTTP log that came with it, then filled in the incident report template.

---

## Section 1: Identify the type of attack that may have caused this network interruption

**One potential explanation for the website's connection timeout error message is:**
One potential explanation is that the web server is being overwhelmed by a SYN flood, a type of Denial of Service (DoS) attack. The server is receiving far more connection requests than it can handle, so it cannot open connections for real visitors and their requests time out.

**The logs show that:**
The logs show that a single unfamiliar IP address, 203.0.113.0, sent a huge number of TCP SYN requests to the web server (192.0.2.1) on port 443. The flood starts at packet 52 (about 3.39 seconds into the capture). Of the 173 packets in the log, 140 come from 203.0.113.0, and 139 of those are SYN requests, all from the same source port 54770. After it completed one handshake (packets 52 to 54), it never sent a web request; it just kept sending new SYNs. At the same time, legitimate visitors start getting errors. For example, 198.51.100.5 receives an HTTP 504 Gateway Time-out at packet 77, and 198.51.100.16, 198.51.100.7, 198.51.100.22 and 198.51.100.9 receive [RST, ACK] responses instead of a connection (packets 73, 80, 85 and 121).

**This event could be:**
This event could be a SYN flood attack, a type of DoS attack in which one attacker sends repeated SYN packets to exhaust the server's resources. Because all of the malicious traffic comes from one IP address, it appears to be a DoS attack and not a Distributed Denial of Service (DDoS) attack.

---

## Section 2: Explain how the attack is causing the website to malfunction

**When website visitors try to establish a connection with the web server, a three-way handshake occurs using the TCP protocol. Explain the three steps of the handshake:**

1. SYN: The visitor's device sends a SYN (synchronize) packet to the web server's port, asking to start a connection. In the log, packet 55 shows 198.51.100.14 sending a SYN to port 443.

2. SYN, ACK: The web server replies with a SYN, ACK (synchronize, acknowledge) packet, which accepts the request and reserves resources for the connection while it waits for the final step. Packet 56 shows this reply.

3. ACK: The visitor's device sends a final ACK (acknowledge) packet to confirm the connection. The connection is now established and data can be exchanged. In packet 58 the visitor sends the ACK, then requests /sales.html in packet 60 and receives HTTP/1.1 200 OK in packet 62.

**Explain what happens when a malicious actor sends a large number of SYN packets all at once:**
When a malicious actor sends a large number of SYN packets all at once, the server responds to each one with a SYN, ACK and sets aside resources for a connection that the attacker never actually uses. If the SYN requests arrive faster than the server can clear them, it runs out of resources for new connections. Legitimate visitors then cannot connect, so their requests time out or get rejected, and the website becomes slow or completely unavailable.

**Explain what the logs indicate and how that affects the server:**
The logs show the server handling normal traffic at first. Visitors 198.51.100.23 (packets 47 to 51) and 198.51.100.14 (packets 55 to 62) complete the handshake and receive their web pages with HTTP 200 OK responses. Starting at packet 52, the attacker at 203.0.113.0 sends SYN requests faster and faster, and the server starts to struggle. Visitor 198.51.100.5 completes a handshake and requests /sales.html at packet 71, but the server returns HTTP/1.1 504 Gateway Time-out at packet 77 (7.33 seconds). Other visitors receive [RST, ACK] packets instead of connections: 198.51.100.16 (packets 73 and 92), 198.51.100.7 (packet 80), 198.51.100.22 (packet 85) and 198.51.100.9 (packet 121 at 19.84 seconds). The server's last response of any kind is packet 124 at 20.81 seconds. From packet 125 to the end of the log at about 51.8 seconds, the only traffic is SYN requests from 203.0.113.0, and the server no longer responds to anyone.

This affects the server because it is too overwhelmed to accept new connections, so legitimate users cannot reach the website and see connection timeout errors. For the travel agency, that means employees cannot access the sales webpage to find vacation packages for their customers, and customers cannot view or book travel packages, which can cost the company sales and damage customer trust. Taking the server offline briefly to recover and blocking 203.0.113.0 with a firewall rule addresses this attack. In the future, the network could be better protected by setting up a firewall or intrusion prevention system (IPS) to detect and drop abnormal floods of SYN packets, rate limiting connection requests, and enabling SYN cookies on the server. Since a future attacker could spoof or change IP addresses, blocking a single IP should not be the only defense.
