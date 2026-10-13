# Support Lab Evidence

## CLIENT01 baseline

Recorded the [baseline screenshot](BASE-001-client01-baseline.png) on October 13, 2026. This records the existing workstation's state before the new support exercises; it is not evidence that SUPPORT01 has been provisioned.

| Check | Observed result |
| --- | --- |
| Workstation and user | CLIENT01; FISCHERLAB\amelia |
| IPv4 address | 10.10.20.100 |
| Gateway | 10.10.20.1 |
| DNS server | 10.10.10.10 |
| DC01 DNS lookup | 10.10.10.10 |
| FS01 DNS lookup | 10.10.10.20 |
| FS01 TCP 445 | TcpTestSucceeded: True |
| Authenticated IT file read | restore-validation.txt returns “FischerLab controlled file recovery test.” |

The capture verifies these checks in Amelia's existing session. It does not independently establish DHCP lease provenance, domain trust health, Group Policy application, or other department permissions.

## Capture guide

For each ticket, capture three to five focused screenshots: the reported symptom or starting state, the diagnostic evidence, the relevant change, and the final verification. More images are useful only when they explain a distinct finding.

Use filenames such as `SUP-002-01-access-failed.png` and link them from the corresponding ticket after review. Preserve command output and error messages in the ticket where practical.

Record the workstation, account context, and date. Use synthetic identities. Exclude passwords, recovery keys, tokens, personal contact details, and unrelated desktop content. Keep raw output private until reviewed.

Do not mark a ticket resolved just because a configuration screen looks correct. Repeat the original user action and retain its outcome. If a problem remains, record the limitation and keep the ticket open.

