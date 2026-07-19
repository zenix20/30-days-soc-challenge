# Day 6: Introduction to Splunk and SIEM

**Topic:** Security Monitoring, correlating failed and successful logins across hosts
**Tools:** Splunk

## Scenario

Given a fixed dataset ("index=mumb_web01_auth", custom time range 2026-03-25 to 2026-03-26) and a warning that not every event had clean field extraction, the task was to quantify failed SSH logins, identify the top attacking IP, find the busiest host by login volume, and trace a specific attack pattern: an IP that brute-forces one account into failure, goes quiet on it, then starts succeeding under different usernames shortly after.

## Questions & Approach

### 1. How many failed SSH password login attempts are in the dataset?

The instructions explicitly warned that field extraction wasn't fully reliable, so every field-based count needed a full-text verification alongside it:

```spl
index=mumb_web01_auth "Failed password"
| stats count
```
→ **808**

```spl
index=mumb_web01_auth Event="Failed password"
| stats count
```
→ 508 (field-based: undercounted, confirming the extraction gap the instructions warned about)

**Answer: 808** (trusting the full-text count, per the instructions)

### 2. Which source IP generated the most failed login attempts?

Rather than trust the "src_ip" field (same reliability concern), extracted the IP directly from raw text:

```spl
index=mumb_web01_auth "Failed password"
| rex field=_raw "from (?<ip>[\d\.]+)"
| stats count by ip
| sort -count
```

**Answer: 185.243.115.10** (500 failed attempts, dominant over the next IP's 300)

### 3. Which host received the highest volume of login activity (successful + failed)?

First attempt used Splunk's indexed "host" field and got flagged as wrong because every event in this index shares the same indexed "host=mumb_web0". That field is constant across the whole index and can't distinguish anything. The actual per-event host lives in a separate extracted field, "extracted_host":

```spl
index=mumb_web01_auth ("Failed password" OR "Accepted password")
| stats count by extracted_host
| sort -count
```

**Answer: vpn-gateway-ny-01** (1,503 combined events, highest of all hosts)

### 4. Tracing the brute-force → credential-reuse pattern

The scenario: one IP hammers a single account with failures, goes quiet on that account, then successful logins start appearing from the same IP under different usernames minutes later. Task: find the **host** of the first suspicious successful login in that pattern.

**Step 1: confirm which account the known attacker IP was brute-forcing:**
```spl
index=mumb_web01_auth "Failed password" "185.243.115.10"
| rex field=_raw "for (invalid user )?(?<user>\S+) from"
| stats count by user
```
→ "michael.smith", 500 failed attempts, confirms this IP's entire brute-force effort targeted one account.

**Step 2: lay out the full timeline (failures AND successes) from that IP, sorted by time, to find the exact transition point:**
```spl
index=mumb_web01_auth ("Failed password" OR "Accepted password") "185.243.115.10"
| rex field=_raw "for (invalid user )?(?<user>\S+) from"
| table _time, extracted_host, user, _raw
| sort _time
```

The timeline showed failures against "michael.smith" ending at "11:26:53" on "vpn-gateway-ny-01", followed by a gap, then the first successful login under a different username (ava.taylor) at "11:30:33" on a **different host entirely**.

**Answer: bastion-us-east-01**

One important detail here: the answer expected was just the **hostname**, not the username, my first submission attempt included the username and was marked wrong until I submitted the host value.
