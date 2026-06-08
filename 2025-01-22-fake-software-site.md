# Incident Report: Fake Google Authenticator Infection

**Source Capture:** [malware-traffic-analysis.net, 2025-01-22 exercise](https://www.malware-traffic-analysis.net/2025/01/22/index.html)  
**Analyst:** Sarvarbek  
**Date Analyzed:** 06/06/2026  
**Tool Used:** Wireshark  

---

## Summary

This report documents a malware infection found during packet capture analysis. A user searched for Google Authenticator, visited a fake website, and downloaded a malicious file. The infected host was identified as `10.1.17.215`, with the hostname `DESKTOP-L8C5GSJ` and the logged-in user `shutchenson`.

The fake domain used in the infection was `google-authenticator.burleson-appliance.net`. The infected computer also communicated with several external IP addresses associated with Cloudflare-hosted infrastructure. Based on the traffic, the most important indicator to block is the fake domain, not only the IP addresses, because Cloudflare IPs can be shared by many legitimate websites.

---

## Environment Overview

The packet capture shows activity from the `10.1.17.0/24` network.

| Asset | IP Address | Details |
|---|---:|---|
| Router / Gateway | `10.1.17.1` | Default network gateway |
| Company Server | `10.1.17.2` | Hostname: `WIN-GSH54QLW48D` |
| Infected Workstation | `10.1.17.215` | Hostname: `DESKTOP-L8C5GSJ` |
| Windows Domain | N/A | `BLUEMOONTUESDAY` |

---

## Key Findings

| Question | Finding |
|---|---|
| Infected computer IP | `10.1.17.215` |
| Infected computer MAC address | `00:d0:b7:26:4a:74` |
| Hostname | `DESKTOP-L8C5GSJ` |
| Logged-in user | `shutchenson` |
| Fake website | `google-authenticator.burleson-appliance.net` |
| External C2-related IPs | `104.21.16.1`, `104.21.32.1`, `104.21.48.1`, `104.21.64.1`, `104.21.80.1`, `104.21.96.1`, `104.21.112.1` |

---

## Timeline Summary

| Step | Activity |
|---|---|
| 1 | User searched for Google Authenticator |
| 2 | User visited a fake Google Authenticator website |
| 3 | Malicious file was downloaded |
| 4 | Workstation `10.1.17.215` became infected |
| 5 | The infected workstation communicated with external infrastructure |

---

## Analysis Details

### 1. Identifying the Infected Host

The infected workstation was identified by reviewing the network traffic and looking for the internal host with suspicious activity. The host `10.1.17.215` was responsible for most of the relevant traffic and was not the router or the company server. This made it the likely infected workstation.

**Wireshark filter used:**

```text
ip.addr == 10.1.17.215
```

![Filtered traffic showing 10.1.17.215 is the busy host](screenshots/q1-infected-host.png)

---

### 2. Identifying the MAC Address

The MAC address was found by selecting a packet sent from the infected workstation and reviewing the Ethernet source address. The source MAC address for `10.1.17.215` was `00:d0:b7:26:4a:74`.

![Source MAC of the infected host](screenshots/q2-mac-address.png)

---

### 3. Identifying the Hostname

The hostname was found in NBNS traffic. Windows systems often announce or register their hostnames on the local network using NBNS. The infected workstation was identified as `DESKTOP-L8C5GSJ`.

**Wireshark filter used:**

```text
nbns
```

![NBNS name registration showing DESKTOP-L8C5GSJ](screenshots/q3-hostname.png)

---

### 4. Identifying the Logged-In User

The username was found in Kerberos authentication traffic. Kerberos login activity included a `CNameString` field. One value ended with a dollar sign, which usually indicates a computer account. The user account without the dollar sign was identified as `shutchenson`.

**Wireshark filter used:**

```text
kerberos
```

![Kerberos CNameString showing the username](screenshots/q4-username.png)

---

### 5. Identifying the Fake Website

The infected host made a DNS request for `google-authenticator.burleson-appliance.net`. Although the domain contains the words `google-authenticator`, the real registered domain is `burleson-appliance.net`, which is not related to Google. This indicates the site was impersonating Google Authenticator.

**Wireshark filter used:**

```text
dns.qry.name contains "burleson"
```

![DNS query for the fake domain](screenshots/q5-fake-domain.png)

---

### 6. Identifying External Malware Communication

The fake domain resolved to multiple external IP addresses beginning with `104.21`. These addresses are associated with Cloudflare infrastructure. Cloudflare is a legitimate service, but attackers can use it to hide the true location of their server and make malicious traffic look more normal.

The observed external IP addresses were:

```text
104.21.16.1
104.21.32.1
104.21.48.1
104.21.64.1
104.21.80.1
104.21.96.1
104.21.112.1
```

![DNS answer listing the seven C2 addresses](screenshots/q6-c2-addresses.png)

---

## False Positives and Ruled-Out Activity

During the investigation, some traffic looked suspicious at first but was ruled out after review.

- The infected workstation communicated with `v20.events.data.microsoft.com`.
- The domain name looked long and unusual, but it is related to normal Microsoft telemetry.
- This activity was not treated as malicious.
- The `104.21.x.x` IP addresses belong to Cloudflare infrastructure.
- Since Cloudflare is used by many legitimate websites, blocking only the IP addresses may cause issues.
- The fake domain is the stronger indicator to block.

---

## Indicators of Compromise

| Type | Indicator |
|---|---|
| Malicious domain | `google-authenticator.burleson-appliance.net` |
| Infected host IP | `10.1.17.215` |
| Infected host MAC | `00:d0:b7:26:4a:74` |
| Infected hostname | `DESKTOP-L8C5GSJ` |
| User account | `shutchenson` |
| External IP | `104.21.16.1` |
| External IP | `104.21.32.1` |
| External IP | `104.21.48.1` |
| External IP | `104.21.64.1` |
| External IP | `104.21.80.1` |
| External IP | `104.21.96.1` |
| External IP | `104.21.112.1` |

---

## Recommended Actions

1. Isolate `10.1.17.215` from the network.
2. Reset the password for the user account `shutchenson`.
3. Block the domain `google-authenticator.burleson-appliance.net`.
4. Review the host for signs of persistence or additional malware.
5. Collect the downloaded file for malware analysis if available.
6. Check other systems for DNS requests to the same fake domain.
7. Reimage the workstation if malware removal cannot be confirmed.
8. Review web filtering controls to help prevent access to similar fake download sites.

---

## Conclusion

The packet capture shows that workstation `10.1.17.215` was infected after the user visited a fake Google Authenticator website. The infected system was identified as `DESKTOP-L8C5GSJ`, and the logged-in user was `shutchenson`.

The most important indicator from this investigation is the fake domain `google-authenticator.burleson-appliance.net`. The external IP addresses are useful for investigation, but the domain is a stronger blocking indicator because the IP addresses are hosted behind Cloudflare and may be shared with legitimate services.
