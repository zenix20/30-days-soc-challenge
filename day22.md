# Day 22: Introduction to Cloud Security

**Topic:** Cloud Security Fundamentals 
**Tools:** Theory only

## Scenario

A theory-focused day on cloud security fundamentals in AWS, framed as a series of short investigative scenarios: given a description of what's being examined (a misconfigured storage setting, a need for change-attribution evidence, a server with a public IP), identify which core AWS service is actually responsible for that resource or data.

## Questions & Approach

### 1. During an AWS assessment, the analyst finds a storage resource where the public-access control value is null. Which AWS service should be investigated?

**Answer: S3 (Amazon S3 / Simple Storage Service)**

A "storage resource" with a public-access-control setting is the textbook S3 bucket scenario. S3 buckets have explicit block-public-access settings and ACLs governing exposure; a null or unconfigured value is one of the most common root causes behind accidental public data exposure incidents in AWS environments.

### 2. An investigator needs to identify who made a configuration change, which API operation was executed, and when it occurred. Which AWS service contains this evidence?

**Answer: CloudTrail**

CloudTrail is AWS's audit-logging backbone, every API call made against an AWS account (console actions, CLI commands, SDK calls) gets recorded with the identity that made it, the specific action taken, the source IP/region, and the timestamp. It's the first place to look for "who did what and when" in essentially any AWS incident investigation.

### 3. The assessment dashboard identifies a virtual server with a non-null public IP address. Which AWS service hosts this resource?

**Answer: EC2 (Elastic Compute Cloud)**

A "virtual server" is AWS's core compute abstraction, an EC2 instance. EC2 instances can be assigned a public IP (either an auto-assigned public IP or a static Elastic IP) for direct internet reachability, making this the natural service to check whenever a "virtual machine with a public IP" comes up in an assessment.
