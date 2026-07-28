# Day 15: Introduction to Digital Forensics and Incident Response

**Topic:** DFIR Fundamentals: Cisco XDR incident review
**Tools:** Cisco XDR 

## Scenario

Day 15 introduced core DFIR principles through a full walkthrough covering how Digital Forensics and Incident Response are interconnected within a SOC, the steps involved in properly containing and preserving evidence, and an overview of standard DF tooling: Autopsy, FTK Imager, Volatility, and Wireshark before moving into a practical review: the order of volatility for evidence collection, and a Cisco XDR incident response tutorial covering a real walkthrough incident to identify scope and the process responsible for the most observable activity.

## Questions & Approach

### 1. Which evidence should be collected first during a live investigation: Memory or Disk?

**Answer: Memory**

This follows the standard order of volatility principle in digital forensics which is the most volatile evidence, data that disappears fastest, like RAM contents, running processes, network connections, and cached credentials needs to be captured before anything else, since it's lost the moment a system is powered off or even just runs long enough to overwrite that memory space. Disk data, by comparison, persists after shutdown and can be imaged later without the same time pressure.

### 2. In the Cisco XDR incident response covered in this tutorial, how many assets were involved?

**Answer: 4**

### 3. In the Cisco XDR incident response covered in this tutorial, which process generated the highest number of observable events?

**Answer: cmd.exe**

cmd.exe generating the highest volume of observables is consistent with a common attacker pattern, using the built-in Windows command shell as a staging point for reconnaissance, chained commands, or launching other tools, which tends to produce a high volume of process-execution and command-line telemetry compared to any single specialized tool.
