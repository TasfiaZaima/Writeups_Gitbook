# Wireshark Filtering for Cybersecurity Analysis

Wireshark is one of the most widely used tools in cybersecurity and network forensics. It allows analysts to inspect packets, understand communication flows, and investigate suspicious or anomalous network behavior.

However, the real value of Wireshark is not just capturing traffic—it is the ability to filter and interpret packets effectively to extract meaningful security insights.

This guide provides a practical and accurate reference for Wireshark filtering in cybersecurity use cases.

***

## 1. Display Filters vs Capture Filters

Wireshark uses two different filtering systems:

### 1.1 Display Filters (Analysis Phase)

Used **after capturing traffic** to inspect and filter packets.

#### Examples

```
http.request
ip.addr == 192.168.1.1
dns.qry.name contains "example"
```

***

### 1.2 Capture Filters (Capture Phase)

Used **before starting a capture** to limit what is recorded. These use **BPF (Berkeley Packet Filter)** syntax.

#### Examples

```
host 192.168.1.10
port 443
net 192.168.1.0/24
```

***

#### Key Insight

Display filters are used more frequently in cybersecurity because they allow deeper post-capture analysis.

***

## 2. Core Wireshark Display Filters for Cybersecurity

***

### 2.1 HTTP Traffic (Unencrypted Traffic Only)

#### Filters

```
http
http.request
http.response
http.response.code == 200
```

#### Use Cases

* Analyzing unencrypted web traffic
* Inspecting API calls in legacy systems
* Debugging application-layer communication

#### Important Note

Most modern traffic uses **HTTPS (TLS encryption)**, so HTTP content is often not visible unless decrypted.

***

### 2.2 DNS Analysis (Domain Monitoring)

#### Filters

```
dns
dns.qry.name
dns.flags.response == 0
dns.flags.response == 1
```

#### Example

```
dns.qry.name contains "example"
```

#### Use Cases

* Investigating domain lookups
* Identifying suspicious DNS activity
* Debugging DNS resolution failures

***

### 2.3 IP Address Filtering

#### Filters

```
ip.addr == 192.168.1.1
ip.src == 192.168.1.10
ip.dst == 8.8.8.8
```

#### Use Cases

* Tracking communication between hosts
* Identifying internal vs external traffic
* Investigating specific endpoints

***

### 2.4 TCP Analysis (Behavior & Performance)

#### Filters

```
tcp
tcp.port == 443
tcp.flags.syn == 1
tcp.flags.ack == 1
tcp.analysis.retransmission
```

#### Use Cases

* Connection behavior analysis
* Packet retransmission investigation
* Network performance troubleshooting

***

### 2.5 TCP Connection Lifecycle

#### Filters

```
tcp.flags.syn == 1
tcp.flags.syn == 1 and tcp.flags.ack == 1
tcp.flags.fin == 1
```

#### Meaning

* SYN → connection initiation
* SYN-ACK → server response
* FIN → connection termination

***

### 2.6 Packet Anomalies & Analysis Flags

#### Filters

```
tcp.analysis.retransmission
tcp.analysis.out_of_order
tcp.analysis.flags
```

#### Use Cases

* Detecting packet loss
* Identifying unstable networks
* Highlighting abnormal TCP behavior

⚠️ Note: These indicate network behavior, not necessarily attacks.

***

### 2.7 ARP & Basic Network Indicators

#### Filters

```
arp
arp.duplicate-address-detected
icmp
```

#### Use Cases

* Detecting ARP conflicts or spoofing indicators
* Observing ICMP diagnostics (e.g., ping)

***

### 2.8 Suspicious Connection Patterns (Context Required)

#### Filter

```
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

#### Possible Meaning

* Repeated SYN attempts may indicate scanning behavior

⚠️ Important:\
This alone is not evidence of an attack. It must be analyzed in context (frequency, pattern, and destination behavior).

***

## 3. TLS / HTTPS Traffic (Modern Reality)

Most modern applications use **TLS encryption (HTTPS)**.

#### Filters

```
tls
tls.handshake
tcp.port == 443
```

#### Key Limitation

* Payload data is encrypted
* HTTP-level inspection is not possible without TLS decryption

***

## 4. HTTP/2 and QUIC (Modern Web Protocols)

Modern web traffic commonly uses:

* HTTP/2 over TLS
* HTTP/3 (QUIC over UDP)

#### Filters

```
http2
quic
udp.port == 443
```

***

## 5. Practical Cybersecurity Use Cases

***

### 5.1 Investigating Slow or Unstable Traffic

```
http && tcp.analysis.retransmission
```

#### Used For

* Packet loss detection
* Retransmission-heavy sessions
* Network instability analysis

***

### 5.2 Investigating Possible Scanning Behavior

```
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

#### Used For

* Detecting repeated connection attempts
* Supporting reconnaissance analysis

***

### 5.3 DNS Investigation

```
dns && dns.qry.name contains "example"
```

#### Used For

* Monitoring domain activity
* Detecting unusual DNS queries
* Investigating malware-like behavior

***

### 5.4 Session Reconstruction

```
tcp.stream eq 0
```

#### Used For

* Viewing full TCP conversation flow
* Analyzing request-response behavior
* Investigating session-level communication

***

## 6. One of the Most Important Wireshark Features

### Follow TCP Stream

**How to use:**\
Right-click packet → Follow → TCP Stream

#### What it does:

Reconstructs full communication between two endpoints.

#### Used For:

* Application behavior analysis
* Request/response inspection
* Incident investigation

⚠️ Limitation:\
Encrypted TLS traffic cannot be read unless decrypted.

***

## 7. Learning Priorities for Cybersecurity Analysts

### Core Skills

* ip.addr
* dns
* tcp.flags.syn
* tcp.analysis.retransmission
* tcp.stream
* http (legacy systems)

### Advanced Skills

* tls
* http2
* quic
* icmp
* arp.duplicate-address-detected

***

## 8. How to Practice Wireshark Effectively

### Practice Workflow

1. Capture live traffic
2. Browse websites
3. Apply filters:
   * dns
   * tcp
   * tls (if available)
4. Analyze:
   * DNS queries
   * TCP behavior
   * Traffic anomalies

Wireshark filtering is not about memorizing syntax; it is about understanding how network protocols behave and how anomalies appear in real traffic.

Once you understand normal communication patterns, Wireshark becomes a powerful tool for:

* Cybersecurity analysis
* Network forensics
* Incident investigation
