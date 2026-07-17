# Day 4: SSH Brute-Force Investigation

**Topic:** Investigating suspicious SSH login failures
**Tools:** Wazuh Indexer (Discover tab)

## Scenario

A server administrator reported multiple suspicious SSH login failures. As the investigating analyst, the task was to work through the `wazuh-alerts-mumb-2026.07` index in Wazuh Indexer's Discover tab to quantify the failed login activity, identify the responsible source IP, and find the most active host.

This was my first time using Wazuh's Discover tab rather than Splunk, same underlying investigative logic, different UI (OpenSearch/Kibana-style Discover interface with DQL query syntax and a clickable field sidebar for instant top-values breakdowns).

## Questions & Approach

### 1. Determine the exact number of failed SSH password login attempts.

**Approach:**

A loose starting query ("sshd AND fail*") returned 1,699 hits, too broad, since it matched on any sshd-related failure text. The dataset had clean structured fields ("process", "authentication_method", "result"), so I tightened the query to use them directly instead of text matching:

```
process:sshd AND authentication_method:password AND result:failure
```

**Answer: 1,481**

### 2. Determine which source IP address generated the highest number of failed login attempts.

**Approach:** With the same filtered query active, clicked the "source_ip" field in the Discover sidebar, Wazuh/OpenSearch shows a "Top 5 values" breakdown with percentages instantly, no additional query needed.

**Answer: 185.198.61.44** (32.4% of failed attempts)

### 3. Determine the hostname with the highest total number of log events.

**Approach:** Cleared the query filter to look at the full index again, then clicked the "hostname" field in the sidebar the same way.

**Answer: NYC-SFTP01**
