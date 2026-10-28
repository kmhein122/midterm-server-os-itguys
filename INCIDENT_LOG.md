# Incident Log

Group: IT Guys

Members: Thet Htoo Naing, Kaung Myat Hein, Kyawt Kay Khine, Ye Zayar Aung, Paing Thukha Kyaw

Company / domain: NovaPeak / novapeak.lab

Evidence note: The excerpts below are actual terminal output retained from the lab and shared in the assistance conversation. The explanations were prepared retrospectively from those outputs with ChatGPT/Codex assistance. Thet Htoo Naing ran the commands and applied the repairs. `session.log` began after the repairs and records final verification only; it is not an original diagnosis recording. The investigation was AI-assisted, rather than an independent diagnosis completed before consulting AI. This differs from the requested diagnosis-first workflow.

---

## Fault 1 — DHCP advertised a public DNS server

- **Symptom (what we observed):**
  ```text
  C9 DHCP advertises 8.8.8.8, expected the company DNS 192.168.19.10
  ```
  The initial checker also reported that the client could not resolve the company web hostname using its leased resolver.
- **Hypothesis (what we suspected and why):**
  DHCP could lease an address successfully while still advertising the wrong resolver. A public resolver would not serve the company's private DNS zone.
- **Proof (the command that confirmed the cause):**
  Relevant excerpt from `cat /etc/dhcp/dhcpd.conf`:
  ```text
  option domain-name-servers 8.8.8.8;
  ```
  The server interface and DHCP interface selection were consistent:
  ```text
  INTERFACESv4="lan0"
  ```
  `ip -br addr show lan0` showed `192.168.19.10/24` on the UP interface.
- **Fix (what we changed):**
  Backed up `/etc/dhcp/dhcpd.conf`, changed the DNS option to `192.168.19.10`, ran `dhcpd -t -cf /etc/dhcp/dhcpd.conf`, and restarted `isc-dhcp-server`.
- **Confirmation (output showing it now works):**
  ```text
  $ grep domain-name-servers /etc/dhcp/dhcpd.conf
          option domain-name-servers 192.168.19.10;
  ```
  The final checker verified the actual client lease and integration:
  ```text
  [   3/3 ] C9   DHCP advertises the company DNS (192.168.19.10)
  [   3/3 ] C11  client resolves www via the leased resolver
  ```
  C11 was confirmed after the DNS and firewall repairs below; changing the DHCP option alone did not establish successful resolution.
- **Found/proved by:** Thet Htoo Naing, using ChatGPT/Codex-assisted interpretation and instructions.

---

## Fault 2 — Forward zone pointed to a nonexistent file

- **Symptom (what we observed):**
  ```text
  $ named-checkconf -z
  zone novapeak.lab/IN: loading from master file /etc/bind/db.MISSING failed: file not found
  zone novapeak.lab/IN: not loaded due to errors.
  _default/novapeak.lab/IN: file not found
  ```
- **Hypothesis (what we suspected and why):**
  The zone declaration referenced the wrong filename, preventing BIND from loading the company's forward zone.
- **Proof (the command that confirmed the cause):**
  `cat /etc/bind/named.conf.local` showed:
  ```text
  zone "novapeak.lab" {
          type master;
          file "/etc/bind/db.MISSING";
  };
  ```
  `ls -l /etc/bind/db.*` showed the actual forward zone file:
  ```text
  -rw-r--r-- 1 bind bind 212 Oct 28 08:54 /etc/bind/db.novapeak.lab
  ```
- **Fix (what we changed):**
  Backed up `named.conf.local` and changed the forward zone file reference to `/etc/bind/db.novapeak.lab`.
- **Confirmation (output showing it now works):**
  Immediately after changing the path:
  ```text
  $ named-checkconf -z
  zone novapeak.lab/IN: loaded serial 2024010101
  ```
  After the subsequent A-record repair and BIND restart, the final client checker reported:
  ```text
  [   2/2 ] C1   named is authoritative for novapeak.lab and reachable from the client.
  ```
- **Found/proved by:** Thet Htoo Naing, using ChatGPT/Codex-assisted interpretation and instructions.

---

## Fault 3 — API hostname had the wrong A record

- **Symptom (what we observed):**
  The initial checker expected the API address `192.168.19.40`, but did not obtain an answer. The zone-loading and reachability faults initially masked the incorrect record.
- **Hypothesis (what we suspected and why):**
  After identifying the zone file, its API record needed to be compared against the expected address. Restoring zone loading alone would not correct an incorrect address inside the zone.
- **Proof (the command that confirmed the cause):**
  Relevant excerpt from `cat /etc/bind/db.novapeak.lab`:
  ```text
  api IN A 192.168.19.1
  ```
  This differed from the checker's expected `192.168.19.40`.
- **Fix (what we changed):**
  Backed up the forward zone, changed the API A record to `192.168.19.40`, and incremented its SOA serial from `2024010101` to `2024010102`. Validated the zone and configuration, then restarted `named`.
- **Confirmation (output showing it now works):**
  ```text
  $ named-checkzone novapeak.lab /etc/bind/db.novapeak.lab
  zone novapeak.lab/IN: loaded serial 2024010102
  OK
  ```
  TCP query, before removing the UDP firewall block:
  ```text
  $ dig +tcp @192.168.19.10 api.novapeak.lab A +noall +comments +answer
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64834
  api.novapeak.lab.       3600    IN      A       192.168.19.40
  ```
  Final checker:
  ```text
  [ 3.0/3 ] C2   all four A records correct
  ```
- **Found/proved by:** Thet Htoo Naing, using ChatGPT/Codex-assisted interpretation and instructions.

---

## Fault 4 — Firewall dropped UDP DNS requests

- **Symptom (what we observed):**
  After loading the repaired zone, TCP DNS returned the correct API address, but a UDP query timed out:
  ```text
  $ dig @192.168.19.10 api.novapeak.lab A +time=2 +tries=1
  ;; communications error to 192.168.19.10#53: timed out
  ;; no servers could be reached
  ```
- **Hypothesis (what we suspected and why):**
  The successful TCP answer showed the repaired record was served. The UDP timeout suggested a transport-specific reachability fault. BIND was already listening on the company address, and `allow-query` permitted queries.
- **Proof (the command that confirmed the cause):**
  Relevant output from `iptables -S`:
  ```text
  -P INPUT ACCEPT
  -A INPUT -p udp -m udp --dport 53 -j DROP
  ```
  This rule explicitly dropped incoming UDP DNS traffic. Before and after firewall snapshots are in `evidence/`.
- **Fix (what we changed):**
  Saved the firewall rules, then removed the specific DROP rule:
  ```bash
  iptables -D INPUT -p udp -m udp --dport 53 -j DROP
  ```
- **Confirmation (output showing it now works):**
  ```text
  $ iptables -S INPUT
  -P INPUT ACCEPT
  ```
  Repeating the UDP query returned:
  ```text
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9551
  api.novapeak.lab.       3600    IN      A       192.168.19.40
  ;; SERVER: 192.168.19.10#53(192.168.19.10) (UDP)
  ```
  The final checker also verified reachability from the client and successful resolution through its leased DNS server.
- **Found/proved by:** Thet Htoo Naing, using ChatGPT/Codex-assisted interpretation and instructions.

---

## Final verification and limitations

The recorded final verification ran configuration checks and the live client checker:

```text
[   2/2 ] C4   named-checkconf -z clean; named-checkzone clean
[   2/2 ] C5   dhcpd -t clean; service running
[   2/2 ] C1   named is authoritative for novapeak.lab and reachable from the client.
[ 3.0/3 ] C2   all four A records correct
[   2/2 ] C3   reverse lookup of 192.168.19.10 returns ns1.
[   3/3 ] C6   client obtained a lease: 192.168.19.104
[ 2.0/2 ] C7   pool+netmask correct
[   2/2 ] C8   gateway advertised correctly
[   3/3 ] C9   DHCP advertises the company DNS (192.168.19.10)
[   1/1 ] C10  company domain advertised
[   3/3 ] C11  client resolves www via the leased resolver
TOTAL: 25.0 / 25
```

The reverse zone, DHCP pool/netmask, gateway, domain option, DHCP interface selection, and BIND listen/query settings required no changes in the observed investigation. Firewall persistence across a reboot was not tested; the evidence establishes the repaired state in the running lab.

