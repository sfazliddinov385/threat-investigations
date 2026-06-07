# Network Traffic Investigations

Real malicious packet captures, investigated end to end. For each one I work out what happened, which machine was hit, who was on it, and what I'd do about it.

Captures come from [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/). The raw `.pcap` files aren't included here. They hold live malicious traffic and aren't mine to share. Each report links back to its source.

## Investigations

| Date | What happened | Threat | Report |
|---|---|---|---|
| 2025-01-22 | A user downloaded a fake Google Authenticator file | Lookalike-site malware with C2 hidden behind Cloudflare | [report](./2025-01-22-fake-software-site.md) |

## How I work each one

- Pull the capture into Wireshark and filter down to what matters
- Identify the infected machine, its user, and its name from DHCP, NBNS, and Kerberos
- Separate the real threat from normal background noise like telemetry and CDNs
- Write it up with the indicators and a containment plan

## Tools

- Wireshark
