# Day 23: Detection Engineering Crash Course

**Tools:** Theory only

## What is Sigma?

**Sigma is a vendor-neutral, open standard for writing detection rules.** The core problem it solves: every SIEM platform (Splunk, Elastic, Microsoft Sentinel, QRadar, and others) has its own query language eg Splunk uses SPL, Elastic uses DQL/Lucene, and so on. If a detection engineer writes a rule to catch, say, suspicious PowerShell execution, they'd normally have to write and maintain that same logic separately in every single platform their organization uses which is a lot of duplicated effort, and a maintenance nightmare whenever the detection logic needs updating.

Sigma fixes this by acting as a **common intermediate format**. A detection is written once as a Sigma rule (a structured YAML file describing what event fields to look for and what values should trigger an alert), and then a converter tool (like the sigma-cli tool, built on the pySigma library) translates that single rule into the actual native query syntax for whatever backend SIEM is being targeted. Write the logic once, deploy it anywhere Sigma has a supported backend.

This makes Sigma genuinely useful for the wider security community too: threat intel teams, researchers, and vendors publish detection rules in Sigma format (there's a large public Sigma rule repository) so that anyone, regardless of which SIEM they run, can pull in a detection for a newly-discovered technique and convert it to their own platform, rather than every vendor/community maintaining separate rule sets in isolation.

## Questions & Approach

### 1. Which vendor-neutral detection format uses YAML and can be converted into rules for different SIEM platforms?

**Answer: Sigma**

### 2. In a Sigma rule, which field is used to identify the executable file being launched?

**Answer: Image**

Sigma rules targeting process-creation telemetry (e.g., built on Sysmon EventID 1 or Windows Security EventID 4688) use Image as the standard field name for the full path of the executable that was launched, this mirrors the actual field name Sysmon itself uses, which Sigma's detection schema is built around for Windows process events.

### 3. Which Sigma modifier checks whether a field value ends with a specified string?

**Answer: endswith**

Sigma supports string-matching modifiers attached directly to a field name in the rule's detection block (written as fieldname|modifier: value). The main string modifiers are:
- contains: substring match anywhere in the value
- startswith: value begins with the given string
- endswith: value ends with the given string

For example, a rule detecting suspicious file extensions on a command line might use CommandLine|endswith: '.ps1' to catch any command ending in a PowerShell script extension, regardless of what path or arguments precede it.
