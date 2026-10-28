# Recorded Repeat Troubleshooting Demonstration

- Group: IT Guys
- Operator: Thet Htoo Naing
- GitHub account: 6705140009-thn
- Scenario: serveros-midterm-lab01
- Company / domain: NovaPeak / novapeak.lab
- Checker group: lab01
- Recording: 2026-10-28 11:10:52–11:21:56 UTC
- Recording duration: 11 minutes 4 seconds
- Recording exit code: 0
- Evidence commit: 783b29a

## Purpose and Prior Knowledge

This is an additional recorded demonstration in a fresh instance of the same lab. The original run had only a final-verification recording. The group already knew the faults and had used AI guidance, so this repeat is not an independent first diagnosis and does not replace the original history.

Initial environment/repository setup and an earlier baseline checker invocation occurred before this recording. The submitted transcript begins before the recorded baseline rerun and before all repairs performed in this fresh instance. It captures the baseline, diagnostic output, explanations, changes, intermediate checks and final validation.

Thet Htoo Naing operated this repeat. No claim is made that all five members operated or attended it. Other members' earlier review contributions remain separately documented.

## Checker Progression

| Stage | Score | Evidence | Interpretation |
| --- | --- | --- | --- |
| Recorded baseline | 11/25 | [Baseline checker](checker-before.txt) | Wrong advertised DNS, zone-loading failure and unreachable client DNS |
| After DHCP DNS correction | 14/25 | [After DHCP](checker-after-dhcp.txt) | C9 passed; DNS/firewall faults remained |
| After forward-zone path correction | 15/25 | [After zone path](checker-after-zone-path.txt) | C4 passed; API data and UDP reachability still needed repair |
| After API A-record correction | 15/25 | [After API](checker-after-api.txt) | Local TCP answer corrected; client checks still masked by UDP drop |
| After targeted firewall correction | 25/25 | [After firewall](checker-after-firewall.txt) | All C1–C11 passed |
| Final saved result | 25/25 | [Final checker](checker-final.txt) | Copy of the successful post-firewall checker result, not a separate invocation |

## Repairs and Evidence

1. DHCP resolver changed from 8.8.8.8 to 192.168.19.10. Syntax validation succeeded, the service restarted, and C9 confirmed the company DNS was advertised.
2. Forward zone declaration changed from db.MISSING to db.novapeak.lab. Zone loading succeeded. A subsequent TCP query exposed the remaining incorrect API address.
3. API A record changed from 192.168.19.1 to 192.168.19.40; serial incremented from 2024010101 to 2024010102. Validation succeeded and TCP returned the corrected answer.
4. The specific INPUT rule dropping UDP port 53 was removed. UDP timed out before removal and returned the authoritative .40 answer afterward. Client integration then passed.

See the [repeat incident log](INCIDENT_LOG.md) for the symptom–hypothesis–proof–fix–confirmation entries.

## Final Verification

[Final verification](final-verification.txt) includes successful BIND configuration/zone checks, DHCP validation, active services, all four authoritative PTR answers and the saved final client-checker result. Local DNS queries are distinct from the client's integration checks: C6–C11 establish the lease and resolution through the DHCP-advertised resolver.

The [configuration diff](config-changes.diff) shows changes to exactly three configuration files: dhcpd.conf, named.conf.local and db.novapeak.lab. The other three collected configuration files required no changes. The [firewall diff](firewall-changes.diff) shows removal of the DROP rule; timestamp and packet-counter changes also appear and are not additional firewall-rule changes.

## Evidence Index

- [Original transcript](session.log) — unedited script recording; terminal control characters and line wrapping are retained.
- [Untouched configuration snapshots](before-configs/) and [repaired snapshots](after-configs/).
- [Diagnostic output before repairs](diagnostics-before.txt).
- [Firewall before](firewall-before.rules) and [firewall after](firewall-after.rules).
- [DHCP reasoning](dhcp-reasoning.txt) and [validation](dhcp-validation-after.txt).
- [Zone reasoning](zone-reasoning.txt) and [validation](zone-validation-after.txt).
- [API reasoning](api-reasoning-before.txt), [validation](api-validation-after.txt), [TCP answer before API repair](api-tcp-after-zone-path.txt) and [TCP answer after repair](api-tcp-after-record-fix.txt).
- [Firewall proof before removal](firewall-proof-before.txt) and [confirmation afterward](firewall-confirmation-after.txt).

The zone explanation was entered before the path correction in the transcript, then saved to its separate reasoning file after the correction; that file explicitly states its save timing. No separate pre-repair zone-proof file is claimed. The original declaration and missing-file error are present in diagnostics-before.txt and the transcript.

## AI Assistance and Limits

ChatGPT/Codex supplied the repeat-run commands, interpretation and wording for the reasoning notes. The notes must not be read as proof of unaided reasoning. This report and the structured incident entries were drafted after the run using the actual saved evidence. The group did not conduct a new two-AI comparison during this repeat; the earlier comparison remains in the root AI_CRITIQUE.md.

This repeat improves coverage of the recorded repair process. It cannot retroactively make the original run diagnosis-first or remove the group's prior knowledge. Reboot persistence, unqueried records and general network security were not independently tested. Credit for the supplementary demonstration is determined by the instructor.

