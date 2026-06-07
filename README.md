# SOC Network Traffic Analysis

This is a set of network traffic investigations. For each one I take a real malicious packet capture, work it the way a Tier 1 SOC analyst would, and write up what happened and what I would do about it.

The captures come from [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/). I do not include the raw `.pcap` files here. They hold live malicious traffic and are not mine to share. Each report links back to where it came from.

## Investigations

| Date | What happened | Threat | Report |
|---|---|---|---|
| 2025-01-22 | A user downloaded a fake Google Authenticator file | Lookalike-site malware with C2 hidden behind Cloudflare | [report](./2025-01-22-fake-software-site.md) |

## What this shows I can do

- Read packet captures in Wireshark and filter down to what matters
- Find the infected machine, its user, and its name from DHCP, NBNS, and Kerberos
- Tell real malicious traffic apart from normal background noise like telemetry and CDNs
- Write up findings as a clear report with a containment plan

## Tools

- Wireshark