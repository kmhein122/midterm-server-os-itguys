# Server OS Midterm — IT Guys

- Group name: IT Guys
- Company / domain: NovaPeak / `novapeak.lab`
- Checker group ID: `lab01`
- Group repository: https://github.com/kmhein122/midterm-server-os-itguys

## Members

| Name | GitHub username | Student ID |
| --- | --- | --- |
| Thet Htoo Naing | 6705140009-thn | 6705140009 |
| Kaung Myat Hein | kmhein122 | 6705140002 |
| Kyawt Kay Khine | 6705140042-lgtv | 6705140042 |
| Ye Zayar Aung | yezayar191004-prog | 6705140012 |
| Paing Thukha Kyaw | Flexxzzzz | 6705140018 |

## Result

The live client checker reported **25.0 / 25**. The client obtained address `192.168.19.104` by DHCP, received company DNS `192.168.19.10`, and resolved `www.novapeak.lab` using that leased resolver. All four A records and the checked reverse lookup passed.

Command:

```bash
python3 -u /root/grading/check_lab.py --group lab01 --phase build | tee checker_result.txt
```

This is the raw technical checker result; incident-log and AI-critique marks are assessed separately.

## Repairs

1. Changed the DHCP DNS option from `8.8.8.8` to `192.168.19.10`.
2. Corrected the forward zone file reference from `/etc/bind/db.MISSING` to `/etc/bind/db.novapeak.lab`.
3. Corrected the `api` A record from `192.168.19.1` to `192.168.19.40`, and incremented the forward zone serial to `2024010102`.
4. Removed the INPUT rule dropping UDP destination port 53.

## Member Review and Contribution Map

The original investigation and repairs were operated by Thet Htoo Naing with ChatGPT/Codex guidance. Subsequent group-supplied verification evidence is organized into the review scopes below. The review documents were drafted with AI assistance; each named reviewer should personally check and understand their document before committing it under their own identity. The scopes below do not reattribute the original repairs to later reviewers.

| Member | Work / review scope | Evidence |
| --- | --- | --- |
| Thet Htoo Naing | Original lab operation, AI-assisted diagnosis and repairs, config collection, checker and final-session evidence | [Incident log](INCIDENT_LOG.md), [configs](fixed-configs/), [checker](checker_result.txt), [session](session.log) |
| Kaung Myat Hein | DHCP settings, service state and live client integration review; repository setup | [DHCP and integration review](evidence/dhcp-integration-review-6705140002.md) |
| Kyawt Kay Khine | Reverse-zone validation and all four PTR answers | [Reverse DNS review](evidence/reverse-dns-review-6705140042.md) |
| Ye Zayar Aung | Forward-zone validation and all four A answers | [Forward DNS review](evidence/forward-dns-review-6705140012.md) |
| Paing Thukha Kyaw | Original/current firewall evidence, listeners and explicit UDP/TCP DNS tests | [Firewall and transport review](evidence/firewall-dns-review-6705140018.md) |

## Submission files

- `fixed-configs/`: six configuration files copied from the repaired running system.
- `checker_result.txt`: final live client checker output.
- `INCIDENT_LOG.md`: symptoms, hypotheses, proofs, fixes and confirmations.
- `AI_CRITIQUE.md`: AI declaration and comparison.
- `session.log`: recorded verification of the repaired system.
- `evidence/firewall-before.rules` and `evidence/firewall-after.rules`: firewall snapshots.
- `evidence/gemini-output.txt`: actual second-AI response.
- Four member review documents linked above: test results, interpretation, limitations and evaluation of relevant AI suggestions.

## Verification Summary

| Verification | Actual result |
| --- | --- |
| Live client checker | 25.0 / 25 |
| Reverse-zone review | Zone valid; all four PTR answers correct and authoritative |
| Forward-zone review | Zone valid at serial 2024010102; all four A answers correct and authoritative |
| DHCP / integration review | Service active; client leased .104; company DNS advertised; C6–C11 passed |
| Firewall / transport review | Original UDP DROP rule absent from current filter INPUT chain; UDP and TCP API queries returned .40 |

Local direct DNS tests and client integration tests have different scopes. The member documents explain those limits; the local queries are not presented as independent client-network tests.

## Additional Recorded Repeat Demonstration

Thet Htoo Naing completed a fresh-instance repeat on 2026-10-28 with a recording from **11:10:52 to 11:21:56 UTC (11 minutes 4 seconds)**. It covers the recorded baseline rerun, diagnostics, AI-assisted explanations, all four repairs, intermediate checks and final verification. Initial setup and an earlier baseline check occurred before this transcript began.

The observed checker progression was **11 → 14 → 15 → 15 → 25 / 25**. The evidence was committed under his full name and account in `783b29a`.

- [Repeat report and evidence index](evidence/repeat-20261028T111047Z/README.md)
- [Repeat incident log](evidence/repeat-20261028T111047Z/INCIDENT_LOG.md)
- [Recorded workflow](evidence/repeat-20261028T111047Z/session.log)
- [Final client result](evidence/repeat-20261028T111047Z/checker-final.txt)
- [Configuration changes](evidence/repeat-20261028T111047Z/config-changes.diff)
- [Firewall changes](evidence/repeat-20261028T111047Z/firewall-changes.diff)

This supplements the original run rather than replacing it. The faults were already known, and ChatGPT/Codex supplied commands and reasoning text during the repeat. It is not an independent first diagnosis. The final checker file is a copy of the successful post-firewall result; the final-verification file displays that saved result alongside additional validation.

## Recording and authorship

AI tools used were ChatGPT/Codex, Gemini, and Muse (Kyaw Gyi). Their respective uses in explanation, diagnostic guidance, comparison and submission auditing are declared in [AI_CRITIQUE.md](AI_CRITIQUE.md). Thet Htoo Naing operated the live lab and ran the repair commands; the AI tools did not operate the live lab.

Thet Htoo Naing operated the lab terminal and applied the repairs with ChatGPT/Codex guidance. In the original run, recording began after the repairs, at the final verification stage. The root `session.log` therefore does not contain the original diagnosis or repair process. Earlier evidence quoted in the original incident log comes from terminal output retained and shared in the assistance conversation. The later repeat transcript linked above separately records the fresh-instance workflow.

### Git Author Identity and Audited History

The Git author name **Charles** belongs to **Thet Htoo Naing**, as confirmed by him. Commits `5609ca2` and `cf8d524` use that author name and are associated by GitHub with his account `6705140009-thn`. Commit `64be606` uses his full name. These names identify the same member, not an additional group member.

At audited head `63557fb`, the account-linked history was:

| Member account | Commits | Interpretation |
| --- | --- | --- |
| 6705140009-thn | 4 | Reports, repairs/config evidence and documentation updates |
| 6705140042-lgtv | 2 | Reverse-DNS review and expanded analysis |
| yezayar191004-prog | 2 | Forward-DNS review and expanded analysis |
| Flexxzzzz | 4 | Firewall review and AI declaration updates; includes one empty commit (487025b) |
| kmhein122 | 3 | Initial repository commit, merge and DHCP/client integration review (2e9f4b0) |

All five member accounts have file-changing contributions in the audited history. Kaung Myat Hein's DHCP review is present in commit `2e9f4b0`. The empty commit adds no evidence of additional work. The table describes the specified historical head; subsequent commits can be checked in GitHub history.

AI was consulted during diagnosis rather than after an independent human diagnosis. The incident log was prepared retrospectively from retained outputs. These process limitations are disclosed in [AI_CRITIQUE.md](AI_CRITIQUE.md). Passing the technical checker does not remove them or establish a final overall grade.

