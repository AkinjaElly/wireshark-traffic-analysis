# Network Traffic Analysis and Security Assessment using Wireshark

## 1. Objective
The objective of this project was to establish a controlled network analysis lab to capture, inspect, and evaluate network traffic using Wireshark and Kali Linux. By generating specific network behaviors (DNS resolution, ICMP diagnostics, secure web traffic, and active reconnaissance), this project aims to analyze protocol mechanisms and identify potential security risks or configurations within a local network infrastructure.

---

## 2. Tools & Environment
* **Host OS:** Windows 10/11 (Running Wireshark `v4.x` for packet capture)
* **Guest OS:** Kali Linux `2025.4` (Running inside Oracle VirtualBox VM for traffic generation)
* **Network Analyzer:** Wireshark
* **Target Assessment Tools:** `ping`, `nslookup`, `curl`, `nmap`

---

## 3. Methodology & Lab Setup
The project was executed in a dual-environment setup where traffic was generated from an isolated Kali Linux virtual machine and captured live on the host interface via Wireshark.

### Traffic Generation Execution (Kali Linux CLI)
![Kali Linux Traffic Generation](./kali-commands.png)

### Live Packet Analysis Interface (Wireshark Capture)
![Wireshark Packet Capture View](./wireshark-capture.png)

---

## 4. Technical Analysis & Observations

### A. ICMP Analysis (Network Diagnostics)
* **Command Executed:** `ping -c 5 8.8.8.8`
* **Observations:** The network diagnostic successfully transmitted and received 5 packets with `0% packet loss`. The average Round-Trip Time (RTT) was recorded at **20.281 ms** (`min/avg/max/mdev = 17.599/20.281/24.388/2.433 ms`). This establishes a baseline for network latency and path reliability between the guest VM and the public resolver.

### B. DNS Analysis (Name Resolution)
* **Packets Isolated:** 110, 124, 125
* **Traffic Flow:** Local Host (`192.168.0.13`) $\leftrightarrow$ Public Nameserver (`41.212.0.100`)
* **Protocol:** DNS (Domain Name System)
* **Analysis:** The capture caught active domain name queries and responses, such as resolving `a.web.tolstoycomments.com` (Packet 124) and `fp.msedge.net` (Packet 110). Because standard DNS operates over UDP port 53 without native encryption, an external eavesdropper can actively reconstruct the host's browsing history by observing these plaintext transactions.

### C. TCP & TLS Analysis (Encrypted Web Traffic & Handshakes)
* **Packets Isolated:** 115, 123, 130, 131
* **Traffic Flow:** External Servers (`128.136.104.39`, `204.79.197.222`) $\leftrightarrow$ Local Host (`192.168.0.13`)
* **Protocol:** TLSv1.2 / TLSv1.3 (Transport Layer Security)
* **Analysis:** Modern secure connections were successfully observed. Packet 123 demonstrates a modern cryptographic handshake beginning via a `Client Hello` utilizing **TLSv1.3**. Packet 130 flags a `Hello Retry Request` for cipher negotiation, followed by an encrypted application exchange. This provides robust protection against session hijacking and payload sniffing.

### D. Active Reconnaissance & Retransmission Drop Analysis
* **Packets Isolated:** 112, 114, 118, 120, 132
* **Traffic Flow:** Local Host (`192.168.0.13`) $\leftrightarrow$ Target Server (`45.33.32.156`)
* **Protocol:** TCP
* **Analysis:** While running an active Nmap stealth scan (`SYN Stealth Scan`) from the Kali machine against `scanme.nmap.org`, the target network infrastructure reacted defensively. Wireshark highlighted an array of deep red and black lines denoting unexpected `[RST, ACK]` flags (Packets 112, 114, 120, 132). This proves the target host or edge firewall actively tore down unauthorized port probes, limiting the scan accuracy as shown by the terminal's retransmission cap warning.

---

## 5. Security Findings & Recommendations

| Finding ID | Vulnerability / Observation | Risk Level | Description | Recommended Mitigation |
| :--- | :--- | :--- | :--- | :--- |
| **SEC-01** | Plaintext DNS Exposure | **Medium** | Outbound DNS requests are sent unencrypted. Attackers executing a Man-in-the-Middle (MitM) attack can map user habits or execute DNS Spoofing/Poisoning. | Implement DNS-over-HTTPS (DoH) or DNS-over-TLS (DoT) on the local gateway or endpoint configurations. |
| **SEC-02** | Reconnaissance Detection & Firewall Drops | **Low/Info** | The Nmap scan generated explicit TCP retransmission flags, confirming that network security controls are dropping unauthorized scanning traffic. | Configure Intrusion Detection Systems (IDS/IPS) to dynamically block and blacklist source IPs exhibiting rapid, multi-port TCP SYN behaviors. |
| **SEC-03** | Use of Legacy TLS Protocols | **Low** | Web sessions observed are relying on TLSv1.2. While secure against basic sniffing, it lacks modern cryptographic optimizations. | Update client and server configurations to enforce TLS 1.3 exclusively to eliminate vulnerable cipher suites. |

---

## 6. Conclusion
This project successfully demonstrates the use of network traffic analysis to validate security baselines. By contrasting unencrypted protocols (DNS) with cryptographically secured streams (TLSv1.2), the lab illustrates how data exposure occurs on a shared medium. Furthermore, observing firewall behavior during an active Nmap scan highlights the importance of defensive logging and proactive traffic monitoring within a Security Operations Center (SOC) framework.
