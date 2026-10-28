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

## Submission files

- `fixed-configs/`: six configuration files copied from the repaired running system.
- `checker_result.txt`: final live client checker output.
- `INCIDENT_LOG.md`: symptoms, hypotheses, proofs, fixes and confirmations.
- `AI_CRITIQUE.md`: AI declaration and comparison.
- `session.log`: recorded verification of the repaired system.
- `evidence/firewall-before.rules` and `evidence/firewall-after.rules`: firewall snapshots.

## Recording and authorship

Thet Htoo Naing operated the lab terminal and applied the repairs with ChatGPT/Codex guidance. Recording began after the repairs, at the final verification stage. `session.log` therefore does not contain the original diagnosis or repair process. Earlier evidence quoted in the incident log comes from terminal output retained and shared in the assistance conversation.

Each member must commit their own actual work using their own Git identity. Further review or documentation contributions should be described accurately in commit messages.

