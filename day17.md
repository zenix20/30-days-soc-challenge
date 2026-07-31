# Day 17: Windows Security Monitoring

**Topic:** Windows Security Monitoring 
**Tools:** Splunk

## Scenario

TechCorp's EDR generated multiple Suspicious Process Creation alerts after several native Windows utilities executed commands containing external URLs, a classic sign of attackers abusing legitimate, pre-installed Windows tools to download and execute malicious payloads rather than dropping obviously suspicious custom malware.

## Questions & Approach

### Getting the schema right first

This dataset (index=mumb-dc01-win-security, host=mumb-dc01) stores raw Windows Event Log data as nested JSON, which needed a couple of exploratory steps before building the real queries:

- host is constant across the whole index (it's the log collector, not the originating machine) the per-event source machine lives in the Computer field instead.
- Event type lives in EventID (not EventCode, which some other Windows datasets in this challenge have used).
- The command line and process name for process-creation events (EventID=4688) are nested: EventData.CommandLine and EventData.NewProcessName.

Confirmed all of this by running EventID=4688 | head 5 and expanding a sample event before building anything else.

### 1. Which Windows utility was used most frequently in commands containing external URLs?

```spl
index=mumb-dc01-win-security EventID=4688 (EventData.CommandLine="*http://*" OR EventData.CommandLine="*https://*")
| stats count by EventData.NewProcessName
| sort -count
```

Results showed four classic LOLBins in play:

**Answer: mshta.exe**

All four of these are well-documented living-off-the-land binaries commonly abused for payload download/execution: certutil for downloading files disguised as certificate operations, bitsadmin for background transfer jobs, regsvr32 for executing remote scriptlets, and mshta for running HTML applications that can pull and execute arbitrary script content.

### 2. Which external host appeared most frequently in the suspicious command lines?

```spl
index=mumb-dc01-win-security EventID=4688 (EventData.CommandLine="*http://*" OR EventData.CommandLine="*https://*")
| rex field=EventData.CommandLine "https?://(?<url_host>[^/\s\"]+)"
| stats count by url_host
| sort -count
```

The results mixed raw IPs and domains, some domains were clearly named to look like infrastructure/services (update-service-cdn.net, dl.dropbox-cdn-files.com, microsoft-helpdesk-online.com) and even an internal-sounding decoy (techcorp-hr-portal.info), while others were bluntly named as attacker infrastructure (cobalt-beacon-c2.ru, beacon-interval-c2.xyz, payload-server-01.cc). The single most frequent value, however, was a raw IP address rather than any of the named domains.

**Answer: 45.33.32.156** (9 occurrences)

### 3. Which system recorded the highest number of process executions containing external URLs?

```spl
index=mumb-dc01-win-security EventID=4688 (EventData.CommandLine="*http://*" OR EventData.CommandLine="*https://*")
| stats count by Computer
| sort -count
```

**Answer: SRV-PROXY01.techcorp.local** (7 occurrences)
