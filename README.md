# SYN Flood Attack Analysis

## Project Overview

This project documents a guided lab from the Google Cybersecurity Certificate. The lab provides a scenario and a Wireshark TCP/HTTP log captured by a security analyst at a travel agency. I reviewed the scenario, analyzed the TCP and HTTP traffic in the log, and wrote an incident report explaining why the company's website was showing connection timeout errors.

The analysis showed that a single IP address was flooding the web server with TCP SYN requests. This is a SYN flood, a type of Denial of Service (DoS) attack, and it left the server unable to respond to legitimate visitors.

---

## Scenario

A travel agency's employees regularly use the company's sales webpage to search for vacation packages for their customers. A monitoring system alerted the security analyst in the scenario to a problem with the web server, and visitors trying to load the site were getting a connection timeout error. The analyst captured traffic with a packet sniffer and saw a large number of TCP SYN requests coming from an unfamiliar IP address. In response, the server was taken offline temporarily so it could recover, and a firewall rule was added to block the attacker's IP address. That block is only a short-term fix, because an attacker can spoof other IP addresses to get around it. My part was to review the provided log, identify the attack, and write up the incident report.

---

## Tools Used

- Wireshark TCP/HTTP log (packet capture provided in the lab as a spreadsheet)
- "How to read a Wireshark TCP/HTTP log" guide from the course
- Google Cybersecurity Certificate incident report template

---

## Key Findings

- **Attacker:** 203.0.113.0, sending from source port 54770 to the web server (192.0.2.1) on port 443.
- **Flood start:** packet 52, about 3.39 seconds into the capture.
- **Volume:** 140 of the 173 packets in the log come from 203.0.113.0, and 139 of those are SYN requests. After one completed handshake (packets 52 to 54), the attacker never sent a web request.
- **Normal traffic before the flood took hold:** 198.51.100.23 (packets 47 to 51) and 198.51.100.14 (packets 55 to 62) completed the handshake and received HTTP 200 OK responses.
- **First errors for real visitors:** an [RST, ACK] to 198.51.100.16 at packet 73 (6.23 seconds) and an HTTP 504 Gateway Time-out to 198.51.100.5 at packet 77 (7.33 seconds), right after its GET request in packet 71.
- **More rejected visitors:** [RST, ACK] responses to 198.51.100.7 (packet 80), 198.51.100.22 (packet 85), 198.51.100.16 again (packet 92), and 198.51.100.9 (packet 121 at 19.84 seconds).
- **Server stops responding:** the server's last response of any kind is packet 124 at 20.81 seconds. From packet 125 to the end of the log (about 51.8 seconds), the only traffic is SYN requests from the attacker.
- **Attack type:** a SYN flood DoS attack. All the malicious traffic comes from one IP address, so it looks like DoS rather than DDoS.

---

## How the Attack Works

**The normal TCP three-way handshake:**
1. **SYN:** the visitor asks the server to open a connection.
2. **SYN, ACK:** the server accepts and reserves resources for the connection.
3. **ACK:** the visitor confirms, and data such as an HTTP request can flow.

**The SYN flood:** the attacker sends SYN after SYN without ever using the connections. Each one makes the server reserve resources, and once those resources run out, the server can't accept new connections from real visitors. Their requests time out or get reset.

---

## Impact

Employees can't access the sales webpage to find vacation packages for their customers, and customers can't view or book travel packages. That can cost the company sales and damage customer trust.

---

## Recommended Mitigations

- Block the attacking IP with a firewall rule as an immediate fix, keeping in mind that attackers can spoof other IPs.
- Use a firewall or intrusion prevention system (IPS) to detect and drop abnormal floods of SYN packets.
- Rate limit incoming connection requests.
- Enable SYN cookies on the web server so it doesn't reserve resources for half-open connections.

---

## Project Documentation

### [Incident Report](incident-report.md)
The completed incident report with my answers to every template question.

### [Incident Report (PDF)](incident-report.pdf)
The completed report in the course template's format.

### [Wireshark TCP/HTTP Log](wireshark-tcp-http-log.xlsx)
The log provided in the lab. A [CSV version](wireshark-tcp-http-log.csv) is included for easy viewing on GitHub.

---

## Skills Demonstrated

- Reading and interpreting Wireshark TCP/HTTP logs
- Identifying a SYN flood DoS attack from packet evidence
- Understanding the TCP three-way handshake and HTTP status codes
- Recommending network-level mitigations
- Incident documentation

---

## What I Learned

- How to follow a single connection through a log by its IP addresses, ports, and flags.
- A normal visit is SYN, SYN-ACK, ACK, then a request and a response. A flood of SYNs with no requests is a red flag.
- HTTP 504 Gateway Time-out and [RST, ACK] responses to legitimate users show a server that is overwhelmed.
- Blocking one IP helps in the moment, but layered defenses are needed because attackers can spoof addresses.

---

## Disclaimer

This is a guided lab completed as part of the Google Cybersecurity Certificate. The company, website, and IP addresses are part of a training scenario, not a real production environment.

This project is based on a scenario from the Google Cybersecurity Professional Certificate (Coursera). The write-up and analysis are my own.
