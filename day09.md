# Day 9: Wazuh SIEM Masterclass

**Topic:** Wazuh SIEM (exact aggregation, dual-field usernames, and brute-force correlation vs. raw attempt volume)
**Tools:** Wazuh Indexer (Discover + Visualize)

## Scenario

Given a fixed agent ("dev-01.corp.local") and a specified absolute time window in "wazuh-alerts-*", the task was to profile SSH activity end-to-end: which account logged in successfully the most, how many unique usernames appeared overall, which source IPs crossed a 50-failed-login brute-force threshold, which IP triggered actual Wazuh brute-force, and the combined failure volume from the two most aggressive IPs.

This module ended up being less about writing new queries and more about **catching two separate mistakes** that silently skewed early answers.
## Questions & Approach

### 1. Which username had the most successful SSH logins?

Base query:
```
agent.name: "dev-01.corp.local" AND rule.groups: "authentication_success"
```

**Approach: Using the Discover sidebar's "Top 5 values" popup.** 

**Answer: ethan.moore**

### 2. How many unique usernames appear across all SSH alerts (successful + failed)?

Base Query:
```
agent.name: "dev-01.corp.local" AND (rule.groups: "authentication_success" OR rule.groups: "authentication_failed")
```
Since usernames were split across "data.dstuser" and "data.srcuser", this required counting unique values in **both** fields and combining, but not by simply adding the two totals, since an overlapping username in both fields would get double-counted.

- "data.dstuser" distinct values: 12 real simulated usernames, **plus** three clear outliers (root, raj, haxlab) sitting far below the main cluster in volume, almost certainly lab/test artifacts rather than in-scope simulated accounts. Also, root was also repeated in "data.srcuser".
- "data.srcuser" distinct values: 12 (admin, backup, support, ubuntu, www, mysql, guest, oracle, postgres, test, ftp, root)

**Answer: 24**

### 3. Flag any source IP with more than 50 failed login attempts as brute_force. How many IPs get flagged?

```
agent.name: "dev-01.corp.local" AND rule.groups: "authentication_failed"
```
Visualized as an exact Terms aggregation on "data.srcip", sorted descending.

**Important catch here:** the time range initially set (05:30–16:30 on a single day) was actually wrong, the challenge's window spanned into the *next day* (05:30 on day 1 through 16:30 on day 2). Re-running with the corrected range surfaced a 4th IP that crossed the 50-attempt threshold, which hadn't shown up under the truncated window.

**Answer: 4**

### 4. Which IP actually tripped the most real Wazuh brute-force alerts (not just raw attempts)?

**Approach: Using the Discover sidebar's "Top 5 values" popup for data.srcip** 

**Answer: 185.243.115.10**

### 5. Combined, how many failed attempts came from the two most aggressive brute-force IPs?

**Approach: Addes the number of attempts of top 2 Using the Discover sidebar's "Top 5 values" popup for data.srcip** 

**Answer: 559**
