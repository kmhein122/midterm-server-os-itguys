# Reverse DNS Verification

- Group: IT Guys
- Reviewer: Kyawt Kay Khine
- GitHub username: 6705140042-lgtv
- Student ID: 6705140042
- Verification time: 2026-10-28 09:37:46 UTC
- Domain: novapeak.lab
- DNS server: 192.168.19.10

## Purpose

Verify that the repaired DNS server loads the reverse zone and returns the expected PTR record for each of the four company addresses.

## Commands Used

```bash
named-checkzone 19.168.192.in-addr.arpa \
  /etc/bind/db.novapeak.lab.rev

for address in 10 20 30 40; do
  echo "Checking 192.168.19.$address"
  dig @192.168.19.10 -x "192.168.19.$address" \
    +norecurse +noall +comments +answer
done
```

The verification output was saved in the lab as:

```text
evidence/reverse-dns-review-6705140042.txt
```

The relevant actual output is also included below so this review contains its own evidence.

## Results

| Address | Expected PTR | Actual PTR | Result |
| --- | --- | --- | --- |
| 192.168.19.10 | ns1.novapeak.lab. | ns1.novapeak.lab. | Pass |
| 192.168.19.20 | www.novapeak.lab. | www.novapeak.lab. | Pass |
| 192.168.19.30 | mail.novapeak.lab. | mail.novapeak.lab. | Pass |
| 192.168.19.40 | api.novapeak.lab. | api.novapeak.lab. | Pass |

## Actual Evidence

The following excerpts preserve the zone-check result and each query's status, flags and answer. EDNS cookie details are omitted.

### Reverse Zone Validation

```text
zone 19.168.192.in-addr.arpa/IN: loaded serial 2024010101
OK
```

### 192.168.19.10

```text
Checking 192.168.19.10
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34702
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; ANSWER SECTION:
10.19.168.192.in-addr.arpa. 3600 IN     PTR     ns1.novapeak.lab.
```

### 192.168.19.20

```text
Checking 192.168.19.20
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21197
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; ANSWER SECTION:
20.19.168.192.in-addr.arpa. 3600 IN     PTR     www.novapeak.lab.
```

### 192.168.19.30

```text
Checking 192.168.19.30
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17829
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; ANSWER SECTION:
30.19.168.192.in-addr.arpa. 3600 IN     PTR     mail.novapeak.lab.
```

### 192.168.19.40

```text
Checking 192.168.19.40
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57786
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; ANSWER SECTION:
40.19.168.192.in-addr.arpa. 3600 IN     PTR     api.novapeak.lab.
```

## Interpretation

Forward DNS maps a hostname to an IP address using an A record. Reverse DNS maps an IP address to a hostname using a PTR record.

The reverse zone passed validation. All four queries returned NOERROR, the authoritative-answer flag aa, and one PTR answer matching the expected company hostname. The +norecurse option requested answers without recursion.

The reverse zone required no changes during this review. This contribution verifies the existing repaired system rather than claiming an additional repair.

## Scope and Limitations

These queries were run from the repaired server against its own DNS address. They establish that the running server serves the four expected PTR records, but do not independently prove reachability from a separate client or correctness of DHCP-advertised settings.

The earlier live client checker, saved in checker_result.txt, separately reported 25.0 / 25, including the checked reverse lookup and successful client resolution through the DHCP-provided DNS server.

## Additional Technical Analysis

### Why the reverse zone name is 19.168.192.in-addr.arpa

For the lab's IPv4 /24 network, the first three address octets are reversed in the reverse-zone name. Address 192.168.19.40 is queried as 40.19.168.192.in-addr.arpa. The zone's record owner 40 therefore maps to api.novapeak.lab. The trailing dot makes the target an absolute DNS name rather than a name relative to the reverse zone.

### Forward and reverse consistency

The review results map .10 to ns1, .20 to www, .30 to mail and .40 to api. The group's separate forward-DNS verification returns those same addresses for the corresponding names. These two evidence sets agree for all four hosts. A syntactically valid reverse zone alone would not establish that agreement; the individual query answers must also be compared.

### Evaluating the AI advice

Gemini suggested querying the reverse mapping for 192.168.19.40. Testing all four addresses gives broader coverage. Gemini also mixed NOERROR and NXDOMAIN in its description of record failures. They are different response codes: these four tests returned NOERROR with actual PTR answers. An absent queried name can return NXDOMAIN, while an existing name without the requested record type may return NOERROR with no matching answer. The response code and answer section must be read together.

### If a future test fails

A zone-check error should lead to inspection of the declared file, syntax and records. A timeout should lead to investigation of reachability and whether the service is answering; it does not by itself prove a missing PTR. A valid authoritative answer with an unexpected hostname points to record data or the zone being queried. Any future failure would require new evidence before making a change.

## AI Assistance

ChatGPT/Codex assisted with preparing the verification commands, interpreting the supplied results, and drafting this review. The evidence above comes from the actual terminal output of the repaired lab session.
