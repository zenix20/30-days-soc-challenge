# Day 1: Introduction to SOC

**Topic:** Introduction to SOC: foundational concepts every analyst starts with
**Tools:** Splunk

## Scenario

Day 1 opened with core SOC concepts: how teams measure detection speed, what comes after triage in the incident response lifecycle, and a hands-on task inside a Splunk lab to identify where Zeek IDS log data originates.

## Questions & Approach

### 1. Which SOC metric measures how quickly a security team identifies a threat?

**Answer:** MTTD (Mean Time to Detect)

This is the standard metric SOC teams track for detection speed distinct from MTTR (Mean Time to Respond/Resolve), which measures how fast a team acts after detection.

### 2. In the SOC incident response process, what comes immediately after Triage?

**Answer:** Investigation

My first instinct was "Containment," but the hint clarified it: "Before containing a threat, analysts first need to understand what actually happened." That reframed the IR flow correctly as:

```
Detection → Triage → Investigation → Containment → Eradication → Recovery → Lessons Learned
```

Triage tells you that something needs attention and roughly how urgent it is, Investigation is where you actually confirm scope and root cause before you start containing anything.

### 3. Access the Splunk Lab and identify the hostname from which the Zeek IDS logs were collected.

**Approach:**
```spl
index=*
```
Ran a broad search across everything, then clicked into the "host" field in the left-hand field sidebar to see all distinct host values without needing to guess the sourcetype name first.

**Answer:** zeek_IDS_02
