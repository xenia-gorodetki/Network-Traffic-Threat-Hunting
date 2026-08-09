# Network-Traffic-Threat-Hunting
## Part 1: Normal Network Baseline (macOS Idle State)

Before hunting for anomalous or malicious activity, it is critical to establish a baseline of a healthy network. A packet capture was performed on a macOS device in an idle state (all user applications closed) on the local Wi-Fi interface (en0). 

The following standard background activities were identified:

* **Encrypted Background Services (DNS & QUIC):** Even in an idle state, the system actively communicates with Google and Apple services. DNS queries to `calendar.google.com`, `help.apple.com`, and `cds.apple.com` were observed.
  
  ![DNS Queries for Background Services](images/dns.png)

  The system calendar synchronizes data using the QUIC protocol, which is a modern, faster replacement for traditional TCP+TLS.
  
  ![QUIC Protocol Traffic](images/quic.png)

* **Time Synchronization (NTP):** Regular automated communication on UDP port 123 was captured. The system utilizes NTP Version 4 (Network Time Protocol) to continuously verify and synchronize accurate time against internet servers.

  ![NTP Time Synchronization](images/ntp.png)

* **Local Network Maintenance (ARP & ICMP):** The local TP-Link router actively maintains network state. It utilizes ARP broadcasts to resolve the MAC addresses of connected devices. Additionally, the router sends ICMP Echo (ping) requests to the host machine, which immediately responds with Echo replies, verifying device availability. The capture also recorded minor packet loss resulting in standard TCP Retransmissions.

  ![ICMP, ARP and TCP Retransmissions](images/tcprts&icmp.jpg)

* **Apple Ecosystem Discovery (mDNS) & Unsuccessful Handshakes (TCP RST):** Multicast DNS traffic querying the `_companion-link._tcp.local` service was frequent. This is expected behavior within the Apple ecosystem, facilitating connection features like Continuity, Handoff, and Universal Control between Mac and iOS devices. 
  
  Simultaneously, a significant volume of TCP packets flagged with `[RST, ACK]` was recorded between local IP addresses. In a baseline state, this occurs when one device attempts a connection on a specific port to verify service availability, but the target device actively refuses (resets) it because the requested service is currently inactive or asleep.

  ![mDNS Queries and TCP RST Packets](images/mdns&rst&ack.png)
