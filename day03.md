# Day 3: Cyber Threat Landscape

**Topic:** Cyber Threat Landscape: persistence mechanisms, the Cyber Kill Chain, and attack types
**Tools:** Conceptual (MITRE ATT&CK / Cyber Kill Chain)

## Scenario

Day 3 moved away from hands-on log work and into threat modeling concepts, how attackers maintain persistence, how the Cyber Kill Chain phases map to real attacker behavior, and common attack categories.

## Questions & Approach

### 1. Which persistence mechanism uses Registry Run Keys?

**Answer:** Registry Run Keys / Startup Folder (MITRE ATT&CK T1547.001)

This one took a few attempts. I initially answered with the technique name itself ("Registry Run Keys"), then tried the parent technique name ("Boot or Logon Autostart Execution") both marked wrong. The actual expected answer was the **Cyber Kill Chain phase** where this kind of persistence gets planted:

**Correct answer: Installation**

The hint "it ensures malware survives reboot" describes what persistence does, but the question was actually asking which **kill chain phase** this activity falls under, not the technique name. Registry Run Keys are a means of achieving persistence, and persistence is established during the **Installation** phase of the kill chain.

### 2. Which Cyber Kill Chain phase includes creating phishing attachments and exploit payloads?

**Answer:** Weaponization

Full Cyber Kill Chain sequence for reference:
```
Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives
```

### 3. Which attack type primarily aims to overwhelm a service with traffic?

**Answer:** DoS/DDoS (Denial of Service / Distributed Denial of Service)
