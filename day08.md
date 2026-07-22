# Day 8: Building SIEM Dashboards

**Topic:** Security Monitoring: dashboard visualization types for common SOC use cases
**Tools:** Splunk (dashboard reference concepts — no live lab for this module)

## Scenario

Unlike most previous days, this module didn't come with a live Splunk lab to query, it tested recognition of the standard chart/visualization types used on typical Splunk security dashboards for common SOC monitoring use cases: an SSH Logs dashboard and a Web Traffic dashboard.

## Questions & Approach

### 1. In the Splunk SSH Logs Dashboard, which chart displays failed login attempts by username?

**Answer:** Bar chart

Bar charts are the standard choice for comparing a count across a categorical field (usernames, in this case), they make it immediately obvious which accounts are taking the most failed-login pressure.

### 2. In the Splunk SSH Logs Dashboard, which chart displays the countries from which possible brute-force attacks originated?

**Answer:** Choropleth map

A choropleth (geographic shading) map is the standard visualization for showing where activity originates by country/region, far more immediately readable than a table of country codes and counts when the goal is spotting geographic concentration of attack traffic.

### 3. In the Splunk Web Traffic Dashboard, which chart compares the top users based on their client IP addresses?

**Answer:** Column chart

Column charts (vertical bars) are typically used here instead of a horizontal bar chart when comparing a moderate number of categories side-by-side, especially when the categories (IP addresses) are shorter labels than something like full usernames or descriptions.

This module was a good reminder that SOC work isn't only about writing the right query — it's also about **choosing (or recognizing) the right way to present the result**. A few defaults that show up constantly across SOC dashboards:

- **Categorical comparisons** (failed logins by user, top attacking IPs, alert counts by severity) → bar or column charts.
- **Geographic distribution** (attack origin by country, traffic by region) → choropleth/world maps.
- **Time-based trends** (login attempts over time, alert volume over a shift) → line or area charts/timecharts.

Being able to glance at an unfamiliar dashboard and immediately guess what each panel is likely showing — before even reading the title — is a small but genuinely useful skill for working efficiently across SIEM tools you didn't personally build.
