# Forward DNS Verification

- Group: IT Guys
- Reviewer: Ye Zayar Aung
- GitHub username: yezayar191004-prog
- Student ID: 6705140012
- Verification time: 2026-10-28 09:45:38 UTC
- Domain: novapeak.lab
- DNS server: 192.168.19.10

## Purpose

Verify that the repaired forward zone validates and that the running DNS server returns the expected A record for all four company hostnames.

## Commands Used

```bash
named-checkzone novapeak.lab /etc/bind/db.novapeak.lab

for host in ns1 www mail api; do
  echo "Checking $host.novapeak.lab"
  dig @192.168.19.10 "$host.novapeak.lab" A \
    +norecurse +noall +comments +answer
done
```

The output was saved in the lab as `evidence/forward-dns-review-6705140012.txt`. Relevant actual output is included below, so this review contains its own evidence.

## Results

| Hostname | Expected A record | Actual A record | Result |
| --- | --- | --- | --- |
| ns1.novapeak.lab | 192.168.19.10 | 192.168.19.10 | Pass |
| www.novapeak.lab | 192.168.19.20 | 192.168.19.20 | Pass |
| mail.novapeak.lab | 192.168.19.30 | 192.168.19.30 | Pass |
| api.novapeak.lab | 192.168.19.40 | 192.168.19.40 | Pass |

## Actual Evidence

These excerpts preserve the zone-check result and each query's status, flags and answer. EDNS cookie details are omitted.

### Forward Zone Validation

```text
zone novapeak.lab/IN: loaded serial 2024010102
OK
```

### ns1.novapeak.lab

```text
Checking ns1.novapeak.lab
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10378
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; ANSWER SECTION:
ns1.novapeak.lab.       3600    IN      A       192.168.19.10
```

### www.novapeak.lab

```text
Checking www.novapeak.lab
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33763
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; ANSWER SECTION:
www.novapeak.lab.       3600    IN      A       192.168.19.20
```

### mail.novapeak.lab

```text
Checking mail.novapeak.lab
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19448
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; ANSWER SECTION:
mail.novapeak.lab.      3600    IN      A       192.168.19.30
```

### api.novapeak.lab

```text
Checking api.novapeak.lab
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7489
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; ANSWER SECTION:
api.novapeak.lab.       3600    IN      A       192.168.19.40
```

## Interpretation

An A record maps a hostname to an IPv4 address. The forward zone passed validation with serial `2024010102`. Each query returned `NOERROR`, the authoritative-answer flag `aa`, and one matching A answer. `+norecurse` requested answers without recursion.

The API record now returns `192.168.19.40`, consistent with the expected address. The earlier incident log documents its original incorrect value, `192.168.19.1`, and the repair. This review verifies the repaired state; it does not claim a new repair or independent discovery of that original fault.

## Scope and Limitations

These queries were made from the repaired server directly to `192.168.19.10`. They confirm that the running DNS server serves the four expected A records. They do not independently verify DHCP lease settings, reverse DNS, or reachability from a separate client.

The earlier live client checker in `checker_result.txt` separately reported `25.0 / 25`, including correct A records and successful client resolution through the DHCP-advertised resolver.

## AI Assistance

ChatGPT/Codex assisted with preparing the verification commands, interpreting the supplied results, and drafting this review. The evidence above comes from the actual terminal output supplied from the repaired lab session.
