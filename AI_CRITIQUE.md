# AI Usage and Critique

Group: IT Guys

Members: Thet Htoo Naing, Kaung Myat Hein, Kyawt Kay Khine, Ye Zayar Aung, Paing Thukha Kyaw

Comparison completed using the actual Gemini response supplied by Thet Htoo Naing. The response is preserved in `evidence/gemini-output.txt`. Gemini's commands were assessed against the existing lab evidence; they were not separately executed as a second repair experiment.

## Part 3a: Declaration (required)

| AI tool | What we used it for |
| --- | --- |
| ChatGPT/Codex | Explained the assignment step by step; interpreted terminal outputs supplied by Thet Htoo Naing; suggested diagnostic and repair commands; helped interpret the checker; prepared documentation from actual retained evidence. During the fresh-instance repeat, also supplied the recording workflow, commands, checkpoint interpretation and wording of the reasoning notes; drafted the repeat report afterward from saved evidence. |
| Gemini | Analyzed the original broken configuration, symptoms and expected network specification in a separate test after the repair; proposed repairs and client verification steps. Its actual response was compared with the retained lab evidence. |
| Muse| Explained the assignment and submission rules from the guideline repo; interpreted the first 11/25 checker screenshot and grouped the failing checks for diagnosis; suggested diagnostic commands for named-checkconf, reachability, and DHCP; checked the Killercoda page read-only (sign-in gate, nothing clicked or logged in); audited the group repo read-only against the guideline, including fixed configs, checker result, incident log, AI critique, session log, evidence, and authorship. Did not operate the live lab, run repair commands, or edit the repository. |

ChatGPT/Codex did not directly operate the live lab. Thet Htoo Naing ran its suggested commands in KillerCoda and supplied the resulting output. Other members' work is attributed according to their actual contributions.

Muse also did not directly operate the live lab or edit the repository. Muse read the public guideline repo and the group's public repo in read-only clones for the audit. Muse did not name db.MISSING, the api .1 address, or the iptables DROP rule before the group found them; it pointed to the config files and diagnostic commands to check. The 3c comparison below covers ChatGPT/Codex and Gemini only; Muse's guidance is declared here and was not run as a separate post-repair comparison like Gemini's.


## Part 3b: Confirm you diagnosed first

We did not complete an independent diagnosis before consulting AI. The early guidance described in Part 3a came from ChatGPT/Codex and Muse while the initial checker results were being interpreted. Thet Htoo Naing ran the diagnostic and repair commands, with step-by-step repair assistance from ChatGPT/Codex. This differs from the assignment's diagnosis-first requirement. We disclose this rather than claim independent discovery.

The fault list below was established through this AI-assisted process: wrong DHCP DNS option, nonexistent forward zone file reference, incorrect API A record, and an INPUT firewall rule dropping UDP port 53. The incident log was prepared retrospectively from retained terminal evidence. Session recording started after the repairs and contains final verification only.

## Part 3c: Putting the AIs to the test

AI #1 received the assignment links and successive screenshots or terminal outputs, including checker symptoms, DHCP and BIND configuration, zone contents and firewall rules. It provided step-by-step guidance and received subsequent results. After the repair, Gemini received a consolidated description of the original broken state: expected addresses, DHCP settings, BIND options and declarations, the zone-loading error, forward/reverse records, the firewall DROP rule and initial client-checker symptoms. Gemini was asked to separate confirmed faults from assumptions, explain repairs, and verify from the client's perspective. The two tools therefore had different interaction histories; this was not an identical blind benchmark.

| Fault (in our words) | We found it? | AI #1 (ChatGPT/Codex) | AI #2 (Gemini) | Was each AI actually right? |
| --- | --- | --- | --- | --- |
| DHCP advertised 8.8.8.8 instead of company DNS | Identified with AI assistance | Suggested advertising 192.168.19.10 | Identified the same incorrect option; suggested .10 and DHCP restart | Both diagnoses match the original config. The applied change was verified by C9 and client integration by C11. Gemini's client-renewal instructions have limitations discussed below. |
| Forward zone referenced db.MISSING | Identified with AI assistance | Suggested the existing db.novapeak.lab file after examining the declaration and file listing | Identified the missing path and suggested the existing filename, a config check and BIND restart | Both diagnoses are supported by the file-not-found error and successful zone loading after the path correction. We used named for the restart; Gemini's bind9 service-name assumption was not separately tested. |
| API A record pointed to .1 instead of .40 | Identified with AI assistance | Suggested .40 and a zone serial increment | Suggested .40, a serial increment and rndc reload | Both proposed the correct data change, verified by the TCP answer and C2. Gemini's rndc reload command was not executed; we validated the zone and restarted named. |
| Firewall dropped UDP destination port 53 | Identified with AI assistance | Suggested removing the specific DROP rule | Identified the same DROP rule and proposed deleting it; also suggested optional UDP/TCP ACCEPT rules | Both diagnoses are supported by the actual rule and UDP timeout before deletion versus a successful answer afterward. The client checker passed after the targeted removal. Extra ACCEPT rules were unnecessary with the observed INPUT ACCEPT policy. |

For each AI suggestion that was WRONG or misleading, describe it and how you verified it was wrong:

- No incorrect technical repair was demonstrated in the retained AI #1 evidence. This does not establish that all AI advice is reliable; the tested suggestions were checked against real command output.
- The checker interruption was a `KeyboardInterrupt` caused by Ctrl+C, not evidence of a Python bug requiring a code change. AI #1 explained this after seeing the traceback.
- A successful local TCP query established that BIND served the corrected API record, but did not prove UDP or client reachability. The UDP before/after test and final client checker supplied that additional evidence.
- Creating a recording after the repair cannot recover the missing original session. AI #1 suggested recording final verification and explicitly describing that limitation.
- Gemini described a record-level failure as a successful `NOERROR` response that also returns `NXDOMAIN`. These are different DNS response codes, not simultaneous statuses. An incorrect A address may be returned with NOERROR; NXDOMAIN means the queried name does not exist. NOERROR with no matching answer is another distinct case. Our corrected API query showed NOERROR with one answer, not NXDOMAIN. This wording could misdirect interpretation of real query output.
- Gemini proposed `dhclient -r && dhclient lan0` from the client. The supplied evidence establishes lan0 as the server interface; it does not establish the client's interface name. The assignment places the client in a separate network namespace. We did not run this command on the server. A manual client test must first identify the client namespace and interface. Our actual client evidence came from the lab checker, including C6, C9 and C11.
- Gemini suggested `nc -zuv` as a UDP reachability test. UDP has no connection handshake, so a port-scan result alone does not prove that BIND answered a valid DNS query. The stronger evidence was the actual `dig` response, including the answer and server/transport information, followed by the live client checker.
- Gemini said an incorrect leased nameserver or gateway means the problem is strictly in dhcpd.conf. That is too broad: a stale lease, a different DHCP server or a client-side override can also affect observed settings. In this lab, the wrong option was actually present in dhcpd.conf, and C9 verified the corrected advertised DNS, so the specific diagnosis was supported.
- Gemini offered unconditional UDP/TCP port-53 ACCEPT rules as an optional best practice. They were not necessary in the observed INPUT ACCEPT configuration after removing the specific DROP rule. We did not add them. A general recommendation to insert rules should account for the intended firewall policy.
- Gemini called the forward/reverse mismatch a split-brain inconsistency. The evidence shows inconsistent records; it does not demonstrate multiple DNS views or divergent DNS servers. The address correction was right, but the label was imprecise.
- Gemini's bind9 service name and rndc reload approach were not independently exercised. They are proposed alternatives, not recorded test successes. Our evidence supports named restart and the final running-system checks.

## Part 3d: Verdict

- **Where our human diagnosis beat the AIs:** We cannot claim an independent human diagnosis beat the AI because the investigation was AI-assisted from the beginning. The human operator supplied direct observations, executed the commands, and returned actual results that the AI could not see on its own. Independent understanding should be demonstrated in each member's real review and explanation.
- **Where an AI genuinely helped:** AI #1 connected the observed failures to specific configuration and runtime causes, suggested separating TCP and UDP tests, and helped organize the evidence. Its repair suggestions were supported by the retained outputs and final 25/25 checker result. Gemini independently analyzed the supplied original-state description and identified the same four faults, giving a useful second assessment of the proposed causes. This was a review of supplied evidence after the repair, not a second live repair run. Its clear fault summary was useful, while its verification details needed closer scrutiny.
- **What we learned about trusting AI to diagnose a live system it cannot see:** Advice must be treated as a hypothesis until tested. A successful DHCP lease did not prove the DNS option was correct. A zone-load success did not prove every record was correct. A local TCP answer did not prove the client could use UDP DNS. The strongest final evidence was the live client obtaining settings by DHCP and resolving through the advertised company resolver. The missing original recording and the diagnosis-first requirement remain process limitations even though the technical checker passed.

## Additional Verification Reviews

The following documents add technical interpretation and critical evaluation using actual post-repair outputs supplied by the group. ChatGPT/Codex helped draft these analyses. They should not be represented as unaided personal writing or as independent pre-AI discovery of the original faults.

| Review scope | Evidence and critical point | Document |
| --- | --- | --- |
| Reverse DNS — Kyawt Kay Khine | All four PTR answers were NOERROR and authoritative. Distinguishes successful answers, NXDOMAIN and NOERROR without a matching answer; extends the suggested single reverse query to all four hosts. | [Reverse DNS review](evidence/reverse-dns-review-6705140042.md) |
| Forward DNS — Ye Zayar Aung | All four A answers matched expected values. Separates zone loading from correct record data, and distinguishes the executed named restart from Gemini's untested rndc alternative. | [Forward DNS review](evidence/forward-dns-review-6705140012.md) |
| DHCP and client integration — Kaung Myat Hein | Service active and C6–C11 passed. Explains why a lease alone is insufficient and why the assumed client lan0 renewal command was not used. | [DHCP review](evidence/dhcp-integration-review-6705140002.md) |
| Firewall and transports — Paing Thukha Kyaw | Current INPUT chain lacks the original DROP rule; explicit UDP and TCP queries both answered correctly. Explains why a UDP scan and extra broad ACCEPT rules were insufficient or unnecessary advice for this evidence. | [Firewall review](evidence/firewall-dns-review-6705140018.md) |

These are reviews of the repaired state, not four additional repairs. Individual Git authorship should correspond to actual review and contributions by each member.

## Supplementary Recorded Repeat: Scope and Evaluation

Thet Htoo Naing completed a fresh-instance repeat on 2026-10-28. The [transcript](evidence/repeat-20261028T111047Z/session.log) runs from 11:10:52 to 11:21:56 UTC and includes the recorded baseline rerun and all subsequent repairs. Initial setup and an earlier unrecorded baseline check occurred before it began. Raw repeat evidence was committed under his account in 783b29a.

The [repeat report](evidence/repeat-20261028T111047Z/README.md) and [incident log](evidence/repeat-20261028T111047Z/INCIDENT_LOG.md) map the evidence to the fault sequence. Scores progressed 11 → 14 → 15 → 15 → 25 / 25. DHCP correction improved C9; zone-path correction improved C4; API correction changed the TCP answer without improving UDP-based client checks; removing the demonstrated UDP DROP rule allowed the final client integration to pass.

ChatGPT/Codex continued to provide commands and interpretation throughout. It supplied the English reasoning text used in the notes, including the DHCP and zone-path explanations, rather than merely correcting independently written student explanations. The API and firewall files explicitly label their reasoning as AI-assisted. These notes and the post-run structured report are not evidence of unaided reasoning. No new Gemini comparison was performed and no additional Muse activity during this repeat is claimed.

The new transcript strengthens evidence of the executed process: it records the wrong settings, repairs, validation and intermediate outcomes, including TCP success versus UDP timeout immediately before firewall repair. The original recording limitation remains true for the original run. Prior knowledge and AI assistance mean the repeat still does not establish an independent first diagnosis or retroactively meet the original diagnosis-first sequence. The instructor determines how this supplementary evidence affects assessment.

