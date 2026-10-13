# SUP-003 Troubleshoot Workstation DNS

**Status:** Resolved — October 13, 2026

## Simulated report

The user cannot reach an internal file server by name.

## Exercise boundary

Create a reversible fault on SUPPORT01 only, using the VMware console so access does not depend on the network being tested. Keep DC01, pfSense, DHCP scope options, and CLIENT01 unchanged. Record the original DNS mode and addresses and keep a local administrator session available before introducing the fault.

## Investigation plan

1. Capture a working baseline: adapter configuration, domain DNS records, and FS01 TCP 445 connectivity by both name and IP. Successful file access should also have been established in the earlier ticket.
2. After the baseline is confirmed, check that SUPPORT01 has no TCP or UDP listener on port 53. If there is no listener, temporarily point Ethernet0 DNS to 127.0.0.1, the workstation itself, to simulate a client DNS misconfiguration. If a listener exists, stop and inspect it before choosing the fault. Do not change the global DHCP options.
3. Clear the client DNS cache, then reproduce the failure. Record the DNS server configuration, the failed name lookup, and whether TCP 445 to 10.10.10.20 still works. Cached names or an alternate resolver may prevent the intended symptom; document what actually happens.
4. Explain which evidence points to name resolution and which evidence tests network reachability. A port test by IP is not proof of successful SMB authentication.
5. Restore the original DNS configuration. If the baseline used DHCP-provided DNS, restore automatic DNS rather than leaving a manual address. Clear the cache again and repeat the name lookup, TCP test, and authenticated IT file read.
6. Verify the final client configuration and record a short explanation for the user.

## Original configuration and rollback

The [healthy baseline capture](../evidence/SUP-003-01-working-dhcp-dns-baseline.png) shows SUPPORT01 with Ethernet0 (interface index 4), IPv4 10.10.20.101, gateway 10.10.20.1, and DNS 10.10.10.10. The netsh output explicitly identifies the DNS server as configured through DHCP. FS01 resolves to 10.10.10.20, and TCP 445 to that IP succeeds.

Restored Ethernet0 DNS server addresses to DHCP-provided values with Set-DnsClientServerAddress -ResetServerAddresses and cleared the DNS client cache. The recovery capture confirms DHCP DNS 10.10.10.10, FS01 resolution, and TCP connectivity. The final user capture confirms the original authenticated file read after recovery.

## Symptom and investigation

The [fault capture](../evidence/SUP-003-02-dns-failure-ip-connectivity.png) shows the guarded script proceeding to set Ethernet0 DNS to 127.0.0.1. After clearing the cache, Resolve-DnsName FS01.corp.fischerlab.test -DnsOnly -NoHostsFile fails with Win32 error 10054, “An existing connection was forcibly closed by the remote host.” TCP 445 to 10.10.10.20 still succeeds from 10.10.20.101. This isolates a name-resolution failure while confirming the tested IP and TCP path remains available; it does not establish authenticated SMB access.

## Root cause and resolution

The controlled fault was a static client DNS override pointing at the workstation itself rather than the domain DNS server. Restored automatic DNS and cleared the cache. The [recovery capture](../evidence/SUP-003-03-dhcp-dns-restored.png) shows DNS supplied through DHCP at 10.10.10.10, FS01 resolving to 10.10.10.20, and TCP 445 succeeding using the server's DNS name.

## Final verification and evidence

DNS configuration and name-based TCP connectivity are restored. The [final user capture](../evidence/SUP-003-04-user-file-access-restored.png) shows whoami returning FISCHERLAB\support.it.test in a normal PowerShell window. Reading \\FS01.corp.fischerlab.test\IT\restore-validation.txt returns “FischerLab controlled file recovery test.” without an error. This completes the original user-action verification after recovery.

## User update

Restored the workstation's automatic DNS configuration. The internal file server now resolves correctly, and access to the IT validation file works again under the affected test account.
