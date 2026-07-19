# Day 7: Splunk SPL Queries

**Topic:** Security Monitoring, separating attacker activity from legitimate account behavior
**Tools:** Splunk

## Scenario

Continuing on the same dataset as Day 6 ("index=mumb_web01_auth", 2026-03-25 to 2026-03-26), this module went a step further: instead of just tracing one attack pattern, it asked for account-level and IP-level aggregate statistics, a brute-force flagging threshold, and most usefully a comparison between an account's attack footprint and its legitimate footprint.

## Questions & Approach

### 1. Which username generated the most total SSH activity (successful + failed combined)?

```spl
index=mumb_web01_auth ("Failed password" OR "Accepted password")
| rex field=_raw "for (invalid user )?(?<user>\S+) from"
| stats count by user
| sort -count
```

**Answer: michael.smith** (807 combined events, far above every other account, consistent with it being both the Day 6 brute-force target and, as this module reveals, a heavily-used legitimate account too)

### 2. How many unique usernames appear anywhere in the dataset?

```spl
index=mumb_web01_auth
| rex field=_raw "for (invalid user )?(?<user>\S+) from"
| stats dc(user) as unique_usernames
```

**Answer: 19**

### 3. Flag any source IP with more than 50 failed logins as brute_force. How many distinct IPs get flagged?

```spl
index=mumb_web01_auth "Failed password"
| rex field=_raw "from (?<ip>[\d\.]+)"
| stats count by ip
| where count > 50
```

Two IPs cleared the threshold: "185.243.115.10" (500 failed attempts) and "185.243.115.30" (300 failed attempts).

**Answer: 2**

### 4. For the account with the most failed logins, how many distinct IPs did it use for successful logins?

First confirmed "michael.smith" was also the account with the most failed logins specifically (500, tied directly to attacker IP "185.243.115.10"), then checked its successful-login IP diversity:

```spl
index=mumb_web01_auth "Accepted password" "michael.smith"
| rex field=_raw "from (?<ip>[\d\.]+)"
| stats dc(ip) as distinct_ips, values(ip) as ip_list
```

**Answer: 77** (almost entirely internal "10.0.1.x" addresses, which reframes the whole picture: "michael.smith" isn't just an attack target, it's a genuinely high-traffic legitimate account that also happened to attract brute-force attempts, rather than a rarely-used account that got compromised outright.

### 5. Combined, how many successful logins came from the two brute-force IPs in Q3?

```spl
index=mumb_web01_auth "Accepted password" ("185.243.115.10" OR "185.243.115.30")
| stats count
```

**Answer: 3**
