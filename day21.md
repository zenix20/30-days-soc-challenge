# Day 21: Vulnerability Management

**Tools:** Theory only

## Scenario

A theory-focused day covering the fundamentals of vulnerability management: how vulnerability severity gets standardized into a comparable score, and what tooling options exist across commercial and open-source vulnerability scanners.

## Questions & Approach

### 1. Which scoring system is commonly used to measure vulnerability severity?

**Answer: CVSS (Common Vulnerability Scoring System)**

CVSS is the industry-standard framework for rating how severe a given vulnerability is, producing a score from 0–10 based on a combination of factors: attack vector (network vs. local), attack complexity, privileges required, user interaction needed, and the resulting impact on confidentiality, integrity, and availability. It's the scoring system referenced almost universally across CVE databases, vendor advisories, and vulnerability scanners.

### 2. Which open-source vulnerability scanner is mentioned as an alternative to Qualys and Tenable?

**Answer: OpenVAS**

OpenVAS (Open Vulnerability Assessment System) is the standard free/open-source option most commonly positioned as the budget-friendly alternative to commercial platforms like Qualys and Tenable (Nessus). It's part of the broader Greenbone Vulnerability Management suite and provides similar core capability, network and host scanning against a maintained vulnerability feed without a licensing cost.
