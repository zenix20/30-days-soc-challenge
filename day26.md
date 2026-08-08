# Day 26: Capstone 3: Suricata IDS Logs Monitoring with Wazuh

**Tools:** Suricata, Wazuh Manager/Agent, Wazuh Dashboard, Kali Linux (attack simulation)

## Scenario

A third infrastructure capstone: integrate Suricata (a network IDS) with Wazuh (a SIEM/XDR platform) so that Suricata's structured JSON event output including DNS and TLS traffic events flows into Wazuh for centralized monitoring, correlation, and custom rule-based alerting.

The intended lab setup:

| Component | Role |
|---|---|
| Suricata IDS | Generates eve.json from network traffic |
| Wazuh Manager | Centralized SIEM for log collection and alert correlation |
| Wazuh Dashboard | Visualization layer for Suricata events |
| Linux host(s) | Hosts Wazuh and/or Suricata |
| Kali Linux | Used to simulate attacks against the monitored network |

## Questions & Approach

### 1. Which Suricata output file is configured in the Wazuh agent to ingest structured network events?

The Wazuh agent's ossec.conf gets a <localfile> block pointing directly at Suricata's JSON event log:
```xml
<localfile>
  <location>/var/log/suricata/eve.json</location>
  <log_format>json</log_format>
</localfile>
```

**Answer: eve.json**

This is Suricata's unified structured-JSON output file rather than logging different event types (alerts, DNS queries, TLS handshakes, flow records) to separate files, modern Suricata deployments consolidate everything into eve.json, with each line tagged by an event_type field (alert, dns, tls, flow, etc.). This is what makes it well-suited for SIEM ingestion: one file, one log format, filterable by event type downstream in Wazuh/Kibana rather than needing to configure multiple separate log sources.

### 2. Which Suricata capture configuration section specifies the network interface being monitored?

**Answer: af-packet**

Suricata's suricata.yaml defines its packet capture method in a dedicated section, af-packet is the standard, most common capture mode on Linux, and it's where the specific network interface(s) to monitor (e.g., interface: eth0) gets declared. (Some Suricata deployments instead use a pcap: section for capture, depending on the chosen capture method but af-packet is the standard default on most modern Linux-based Suricata installs.)
