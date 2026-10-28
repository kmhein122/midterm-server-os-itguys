# Firewall and DNS Transport Verification

- Group: IT Guys
- Reviewer: Paing Thukha Kyaw
- GitHub username: Flexxzzzz
- Student ID: 6705140018
- Verification time: 2026-10-28 09:48:11 UTC
- DNS server: 192.168.19.10
- Test hostname: api.novapeak.lab

## Purpose

Review the saved original firewall rule, inspect the current INPUT chain and DNS listeners, and verify that the repaired server returns the expected API address over both UDP and TCP. This review performs read-only verification and does not apply additional firewall changes.

## Commands Used

```bash
grep -E '^:INPUT |^-A INPUT ' evidence/firewall-before.rules
iptables -S INPUT
iptables -L INPUT -n -v --line-numbers
ss -lntup | grep ':53'

dig @192.168.19.10 api.novapeak.lab A \
  +notcp +ignore +time=2 +tries=1 \
  +norecurse +noall +comments +answer +stats

dig @192.168.19.10 api.novapeak.lab A \
  +tcp +time=2 +tries=1 \
  +norecurse +noall +comments +answer +stats
```

The combined output was saved in the lab as `evidence/firewall-dns-review-6705140018.txt`. Relevant actual excerpts are included below so this review contains its own evidence. Unrelated listener entries and EDNS cookies are omitted.

## Results

| Check | Observed result | Assessment |
| --- | --- | --- |
| Saved original firewall | UDP destination port 53 DROP rule present | Original fault confirmed in saved evidence |
| Current INPUT chain | ACCEPT policy; no listed rules | Original INPUT DROP rule absent |
| Company DNS UDP listener | named bound to 192.168.19.10:53 | Pass |
| Company DNS TCP listener | named listening on 192.168.19.10:53 | Pass |
| UDP query | NOERROR, aa, A record 192.168.19.40 | Pass |
| TCP query | NOERROR, aa, A record 192.168.19.40 | Pass |

## Actual Evidence

### Saved Original Firewall

```text
Original firewall INPUT rules
:INPUT ACCEPT [1152:150580]
-A INPUT -p udp -m udp --dport 53 -j DROP
:INPUT ACCEPT [0:0]
```

The grep command selects matching INPUT lines across the saved iptables tables without displaying table headers. The two policy declarations should not be interpreted as two active INPUT chains in a single table. The relevant original fault is the explicit UDP port 53 DROP rule.

### Current Firewall

```text
Current firewall INPUT rules
-P INPUT ACCEPT
Chain INPUT (policy ACCEPT 10573 packets, 13M bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

The inspected current filter INPUT chain has an ACCEPT policy and no listed rules. The original DROP rule is absent. No extra ACCEPT rules were added during this review.

### Company DNS Listeners

Relevant rows from the actual socket listing:

```text
udp   UNCONN 0      0                           192.168.19.10:53         0.0.0.0:*    users:(("named",pid=3001,fd=25))
tcp   LISTEN 0      10                          192.168.19.10:53         0.0.0.0:*    users:(("named",pid=3001,fd=26))
```

### UDP DNS Query

```text
UDP DNS query
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33292
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; ANSWER SECTION:
api.novapeak.lab.       3600    IN      A       192.168.19.40

;; Query time: 1 msec
;; SERVER: 192.168.19.10#53(192.168.19.10) (UDP)
;; WHEN: Wed Oct 28 09:48:11 UTC 2026
;; MSG SIZE  rcvd: 123
```

### TCP DNS Query

```text
TCP DNS query
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51912
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; ANSWER SECTION:
api.novapeak.lab.       3600    IN      A       192.168.19.40

;; Query time: 1 msec
;; SERVER: 192.168.19.10#53(192.168.19.10) (TCP)
;; WHEN: Wed Oct 28 09:48:11 UTC 2026
;; MSG SIZE  rcvd: 123
```

## Interpretation

DNS can use both UDP and TCP. The original firewall rule specifically dropped UDP port 53, so a successful TCP query alone would not demonstrate that UDP DNS worked. The earlier incident log records the UDP timeout before the rule was removed and the successful UDP response afterward.

In this review, both transports returned the expected API A record with NOERROR and the authoritative-answer flag aa. The SERVER lines explicitly identify UDP and TCP. The UDP command used +notcp and +ignore to prevent a TCP retry after a truncated response. The socket listing also showed named on the company address for both transports.

These results verify the existing repair. They do not claim that Paing Thukha Kyaw discovered or removed the original rule.

## Scope and Limitations

The DNS queries were made from the repaired server to its own company address. They establish successful local DNS exchanges over both transports, but do not independently test the path from a separate client. The earlier live client checker in checker_result.txt provides separate evidence of client reachability and resolution through the DHCP-advertised resolver, with a total of 25.0 / 25.

The current firewall inspection covered the filter INPUT chain, not every possible packet-filtering mechanism. Actual query responses provide behavioral evidence for these specific tests. Persistence after reboot and firewall behavior on other network paths were not tested. The current ACCEPT policy is an observation of this lab, not a general firewall-policy recommendation.

## Additional Technical Analysis

### A listener does not prove a successful DNS exchange

The socket listing shows that named has bound port 53 over both transports. A firewall can still drop packets before they reach those sockets. The original INPUT rule targeted UDP only, explaining why TCP and UDP tests needed to be separated. The saved rule plus the earlier timeout/success comparison supports the repair diagnosis more strongly than the listener listing alone.

### Why the query transport is explicit

The UDP test uses +notcp and +ignore, and its SERVER line reports UDP. The TCP test uses +tcp and its SERVER line reports TCP. Each returns an actual DNS answer with NOERROR, aa and the expected .40 address. This distinguishes application-level success from merely seeing an open socket or a port-scan message. The measured 1 ms response times describe these two queries only; they are not a DNS performance benchmark.

### Evaluating the AI advice

Both tools correctly identified the UDP DROP rule. Gemini proposed nc -zuv as a reachability test, but UDP has no connection handshake, and a scan result alone is not proof of a valid DNS response. The dig answer and explicit transport line provide stronger evidence for these tests.

Gemini also suggested inserting broad UDP and TCP ACCEPT rules as an optional step. The observed filter INPUT chain already has an ACCEPT policy after the specific DROP rule was removed. No extra ACCEPT rules were needed to achieve the verified lab behavior. Firewall changes should address demonstrated causes within the intended policy rather than add rules without a requirement.

### Historical evidence and current evidence

The saved before snapshot establishes that the original rule existed. Current iptables output establishes that it is absent from the inspected chain now. Earlier incident evidence shows the change from UDP timeout to a successful answer. This review retests the repaired state; it does not reintroduce the fault or claim a fresh before/after experiment.

## AI Assistance

ChatGPT/Codex assisted with preparing the verification commands, interpreting the supplied results, and drafting this review. The evidence above comes from the actual terminal output supplied from the repaired lab session.
