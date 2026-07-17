# Day 2: Security Monitoring

**Topic:** Security Monitoring: telemetry types, event IDs, and log sources
**Tools:** Splunk

## Scenario

Day 2 covered the building blocks of security monitoring: the difference between metrics and event-based telemetry, standard Windows Event IDs, key Linux log file locations, and a hands-on SSH log investigation in Splunk.

## Questions & Approach

### 1. Which security telemetry type records CPU usage, error rates, and traffic as numeric values instead of event records?

**Answer:** Metrics

Metrics are numeric, time-series measurements (CPU, error rate, throughput) as opposed to logs/events, which are discrete records of things that happened. This distinction matters for SOC tooling, metrics feed dashboards and threshold-based alerting, while event logs feed correlation and forensic investigation.

### 2. In the "Same Event, Different View" example, what is the Windows Event ID used for a failed logon?

**Answer:** 4625

### 3. Which Linux log file is specifically mentioned for authentication events?

**Answer:** /var/log/auth.log

### 4. Access the Splunk Lab and determine how many unique usernames appear in the SSH log entries for host="mumb_web01".

**Approach:**

The SSH log data was JSON with the username embedded inside a raw `message` field (not a dedicated field), so this needed a `rex` extraction:

```spl
index=* host=mumb_web01
| rex field=message "for (invalid user )?(?<u1>\S+) from"
| rex field=message "Invalid user (?<u2>\S+) from"
| eval user=lower(coalesce(u1, u2))
| stats dc(user) as unique_usernames, values(user) as usernames
```

Key debugging step: the JSON payload itself contained an internal `host` field (e.g., `bastion-us-east-01`) that was different from the Splunk-indexed `host` value. Filtering on the indexed host field (confirmed via the event's metadata sidebar, not the JSON body) was what got real results.

**First pass returned 19** but that count included `admin` as if it were a distinct username, when the challenge intended only human user accounts. Excluding it brought the correct count to:

**Answer:** 18
