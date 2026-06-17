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

1.  **Traffic Generation:** Initiated network diagnostic tools on the Kali Linux machine to target both public infrastructure and specific testing environments (`scanme.nmap.org`).
2.  **Live Capture:** Promiscuous/Standard capture mode was enabled on the active Wi-Fi/Ethernet interface to record inbound and outbound frames.
3.  **Protocol Filtering:** Applied specific Wireshark display filters (`dns`, `tls`, `tcp`, `icmp`) to isolate and dissect targeted handshakes and packet headers.

---

## 4. Technical Analysis & Observations

### A. ICMP Analysis (Network Diagnostics)
* **Command Executed:** `ping -c 5 8.8.8.8`
* **Observations:** The network diagnostic successfully transmitted and received 5 packets with `0% packet loss`. The average Round-Trip Time (RTT) was recorded at **20.281 ms** (`min/avg/max/mdev = 17.599/20.281/24.388/2.433 ms`). This establishes a baseline for network latency and path reliability between the guest VM and the public resolver.

### B. DNS Analysis (Name Resolution)
* **Packets Isolated:** 13124, 13125
* **Traffic Flow:** Local Host (`192.168.0.13`) $\rightarrow$ Public Nameserver (`41.212.0.100`)
* **Protocol:** DNS (Domain Name System)
* **Analysis:** The capture caught raw outbound DNS queries. Because standard DNS operates over UDP port 53 without native encryption, the requested domain strings and destination IP addresses are fully visible in plaintext within the packet byte-stream.

### C. TCP & TLS Analysis (Encrypted Web Traffic)
* **Packets Isolated:** 13115, 13116, 13117
* **Traffic Flow:** External Server (`128.136.104.39`) $\leftrightarrow$ Local Host (`192.168.0.13`)
* **Protocol:** TLSv1.2 (Transport Layer Security)
* **Analysis:** Application layer traffic captured from this stream shows an established TLSv1.2 cryptographic handshake. While the initial setup parameters are visible, the actual payload data is obscured via symmetric encryption, mitigating sniffing vulnerabilities.

### D. Active Reconnaissance & Retransmission Drop Analysis
* **Packets Isolated:** 13114, 13118, 13120
* **Traffic Flow:** Local Host (`192.168.0.13`) $\rightarrow$ Target Server (`45.33.32.156`)
* **Protocol:** TCP
* **Analysis:** During the execution of an Nmap port scan against `scanme.nmap.org`, Wireshark flagged several TCP packets in red/black. This visual warning denotes a high rate of unacknowledged TCP SYN packets and subsequent timeouts. The target host dropped connections actively, prompting the terminal warning: *“giving up on port because retransmission cap hit (10).”* This indicates an active firewall or rate-limiting mechanism on the receiving end.

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
