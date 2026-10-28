# DHCP and Client Integration Verification

- Group: IT Guys
- Reviewer: Kaung Myat Hein
- GitHub username: kmhein122
- Student ID: 6705140002
- Verification time: 2026-10-28 09:46:56 UTC
- Domain: novapeak.lab
- Server interface: lan0
- Company DNS: 192.168.19.10

## Purpose

Verify the repaired DHCP configuration, service state and actual client behavior. A valid configuration alone is insufficient: the client must obtain a lease and resolve a company hostname using the DNS server advertised by DHCP.

## Commands Used

```bash
cat /etc/dhcp/dhcpd.conf
cat /etc/default/isc-dhcp-server
ip -br addr show lan0
dhcpd -t -cf /etc/dhcp/dhcpd.conf
systemctl is-active isc-dhcp-server
python3 -u /root/grading/check_lab.py --group lab01 --phase build
```

The relevant actual terminal output is included below so this review is self-contained and does not depend on a separate raw .txt file.

## Results

| Check | Observed result | Assessment |
| --- | --- | --- |
| DHCP configuration validation | No configuration errors reported; checker C5 passed | Pass |
| DHCP service | active | Pass |
| DHCP interface selection | INTERFACESv4="lan0" | Pass |
| Server interface | lan0 UP with 192.168.19.10/24 | Pass |
| Subnet and pool | 192.168.19.0/24; 192.168.19.104–192.168.19.144 | Pass |
| Gateway | 192.168.19.1; checker C8 passed | Pass |
| Advertised DNS | 192.168.19.10; checker C9 passed | Pass |
| Domain | novapeak.lab; checker C10 passed | Pass |
| Actual client lease | 192.168.19.104; checker C6 passed | Pass |
| Client DNS integration | Client resolved www through its leased resolver; C11 passed | Pass |
| Complete live checker | 25.0 / 25 | Pass |

## Actual Evidence

### DHCP Configuration

```conf
option domain-name "novapeak.lab";
default-lease-time 3600;
max-lease-time 7200;

subnet 192.168.19.0 netmask 255.255.255.0 {
        range 192.168.19.104 192.168.19.144;
        option routers 192.168.19.1;
        option domain-name-servers 192.168.19.10;
}
```

### Interface Selection and Address

```text
INTERFACESv4="lan0"
INTERFACESv6=""
lan0@if4         UP             192.168.19.10/24 fe80::904c:9eff:fe8f:df88/64
```

### Configuration Test and Service State

```text
Internet Systems Consortium DHCP Server 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
active
```

The final `active` line is the result of `systemctl is-active isc-dhcp-server`. The live checker also independently reported a clean DHCP configuration test and a running service.

### Live Client Checker

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
  --------------------------------------------------
  TOTAL: 25.0 / 25
```

## Interpretation

DHCP distributes the client's network settings. In this configuration, the pool supplies client addresses, option routers supplies the gateway, option domain-name-servers supplies the company resolver, and option domain-name supplies the company domain.

The configured DNS address is the repaired company server, `192.168.19.10`. The incident log records its original incorrect value, `8.8.8.8`. This review verifies the repaired state and does not claim discovery or repair of the original fault.

C6 demonstrates an actual client lease rather than merely a syntactically valid pool definition. C9 checks that the company DNS is advertised. C11 then connects DHCP and DNS by verifying that the client resolves the company web hostname through its leased resolver. Together, these checks support the lab's central goal.

## Scope and Limitations

The commands were run in the existing repaired lab. Client behavior was evaluated by the provided live checker rather than a separately authored client test. Advertising a gateway does not by itself establish Internet connectivity, which was not separately tested. These results verify the running lab state; they do not establish behavior after reboot or a fresh deployment.

## Additional Technical Analysis

### A lease can succeed while DNS integration fails

The original checker showed that a client could obtain 192.168.19.104 even while DHCP advertised 8.8.8.8. Successful address allocation therefore did not prove that the complete lease settings supported company DNS. The repaired configuration advertises 192.168.19.10. C9 verifies the advertised resolver and C11 verifies the client's use of that resolver for company-name resolution.

### Reading the subnet settings

The 255.255.255.0 netmask defines this lab's /24 subnet. The .104 through .144 pool lies within it, while the server .10 and gateway .1 are outside that allocation pool. The default and maximum lease durations are 3600 and 7200 seconds. This review observes those configured durations; it did not separately measure lease expiration or renewal timing.

### Evaluating the AI advice

Both AI tools correctly identified the public DNS option as inconsistent with the lab's requirement. Gemini suggested renewing the client with dhclient on lan0, but the supplied evidence establishes lan0 as the server interface, not the client's interface. The lab client runs in a separate network namespace. Those suggested renewal commands were not run on the server; the provided checker supplied the actual client test instead.

Gemini also suggested that an incorrect leased nameserver or gateway strictly identifies a dhcpd.conf problem. That is too broad without checking the actual environment: a stale lease, another DHCP server or a client override can affect observed settings. Here, the incorrect option was directly visible in the original configuration and the final checker verified the corrected advertised resolver.

### Evidence beyond a local DNS query

A command that directly specifies @192.168.19.10 bypasses the question of which resolver DHCP advertised. C11 is useful because it tests resolution through the leased resolver. The final result combines address acquisition, advertised settings and actual name resolution, rather than assuming integration from separate configuration files.

## AI Assistance

ChatGPT/Codex assisted with preparing the verification commands, interpreting the supplied results, and drafting this review. Configuration excerpts and checker results above come from the actual terminal output supplied from the repaired lab session.
