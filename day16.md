# Day 16: Wireshark Crash Course

**Topic:** Network Forensics
**Tools:** Wireshark

## Scenario

Given a packet capture (PCAP) file, the task was to analyze traffic from client 192.168.1.6: count how many concurrent HTTPS sessions it opens to a single local server, identify the destination port of a specific external connection without decrypting any payload, and fingerprint the client's likely operating system purely from consistent IP header behavior.

## Questions & Approach

### 1. The client 192.168.1.6 opens many simultaneous TCP sessions to a single local server on port 443. Count how many distinct concurrent HTTPS sessions the client establishes to this server.

**First approach**: filtered for TCP SYN packets (the start of each new handshake) on port 443 from the client:
```
ip.src == 192.168.1.6 && tcp.port == 443 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```
This returned **42**, which turned out to be wrong, it wasn't scoped to the single local server specifically, so it was counting SYNs toward every port-443 destination the client talked to, local and remote alike.

**Correct approach**: used **Statistics → Conversations → IPv4 tab** to first identify which destination actually qualifies as "a single local server." Scanning the conversation list, one destination stood out as the only private/local IP among a list of otherwise public IPs, and it dominated the traffic volume by a wide margin: 192.168.1.150, with 6,186 packets versus low double-digit packet counts for every other conversation.

[Screenshot: Conversations IPv4 tab identifying 192.168.1.150 as the local server]

With the correct destination isolated, switched to the **TCP tab** in Conversations, which lists each distinct TCP stream as its own row.

**Answer: 39** distinct concurrent HTTPS sessions between 192.168.1.6 and 192.168.1.150

### 2. There's a connection from the client to 216.128.142.200 on TCP port Without decrypting anything, Identify that port number.

TCP port numbers live in the cleartext TCP header regardless of what protocol/encryption is running on top of the connection — this is exactly why the question specifies "without decrypting anything": port numbers are visible on the wire even for fully encrypted traffic like TLS, since encryption only protects the payload, not the header fields used for routing/session identification.

```
ip.addr == 216.128.142.200
```
Checked the TCP conversation for this IP directly in the packet details pane.

**Answer: 9997**


### 3. Examine the IP packets sent by host 192.168.1.6 throughout this capture. Based on a consistent field you can observe in the IP header, identify what operating system this host is most likely running.

```
ip.src == 192.168.1.6
```
Checked the **Time To Live (TTL)** field in the IP header across multiple packets from this host. TTL values cluster around OS-specific defaults, since different operating systems ship with different default starting TTLs:
- ~64 → Linux/Unix/macOS
- ~128 → Windows
- ~255 → Some routers/older Unix systems

The consistent TTL value observed matched the Windows default.

**Answer: Windows**

P/IP header fields (ports, TTL, flags) are always visible in cleartext, independent of whatever encryption protects the payload above them — a genuinely useful fact for any investigation involving encrypted traffic, since metadata analysis doesn't require breaking the encryption at all.
