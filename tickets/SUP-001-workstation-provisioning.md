# SUP-001 Provision a Windows Workstation

**Status:** Resolved — workstation provisioning acceptance checks verified on October 13, 2026

## Request

Prepare SUPPORT01 for a simulated IT employee with domain sign-in, appropriate department access, and applied workstation policies.

## Acceptance criteria

- Unique computer name and DHCP lease on OFFICE.
- Domain DNS resolution and FS01 TCP 445 connectivity.
- Membership in corp.fischerlab.test and computer object in the existing Workstations OU.
- Successful sign-in as an authorized IT employee.
- Applicable user and workstation policies inspected.
- IT test-file read succeeds; an Accounting share access check records the expected authorization boundary.

## Execution checklist

1. Establish healthy baseline connectivity on CLIENT01, then prepare a clean SUPPORT01 workstation on VMware VMnet3 (OFFICE).
2. Capture SUPPORT01 hostname, Windows edition, IPv4 lease, gateway, and DNS.
3. Verify DC01 and FS01 resolve through the configured domain DNS; test FS01 TCP 445.
4. Join corp.fischerlab.test through Windows computer-name/domain settings using an existing authorized lab administrator. Enter credentials interactively, not in a command or document. Restart.
5. On DC01, locate SUPPORT01 in Active Directory Users and Computers and move it to the existing FischerLab Workstations OU if necessary. Verify its placement rather than creating a second OU.
6. On SUPPORT01, sign in as an existing IT test employee. Run `whoami`, inspect domain membership, and run `gpresult /r` in that user session. Apply `gpupdate /force` if needed and follow any restart/sign-out prompt.
7. Read the existing IT validation file at \\FS01.corp.fischerlab.test\IT\restore-validation.txt. Test the Accounting share separately; preserve its exact path and outcome.
8. From an elevated Windows PowerShell session on SUPPORT01, run `Test-ComputerSecureChannel -Verbose` without `-Repair` to check trust. This result and the ordinary employee access tests serve different purposes.

## Actual observations

Recorded lab screenshots on October 13, 2026:

- Initial workstation name was DESKTOP-1EF5QFH. Its IPv4 address was 10.10.20.101, gateway 10.10.20.1, and DNS server 10.10.10.10.
- After the rename and restart, `hostname` returns SUPPORT01 and `whoami` returns `support01\support`, confirming the local account context.
- DC01.corp.fischerlab.test resolves to 10.10.10.10; FS01.corp.fischerlab.test resolves to 10.10.10.20.
- FS01 TCP 445 test returns True from source 10.10.20.101.

The subsequent [domain sign-in capture](../evidence/SUP-001-03-domain-membership-sign-in.png) shows `whoami` returning `fischerlab\amelia` and Win32_ComputerSystem returning SUPPORT01, corp.fischerlab.test, and PartOfDomain True. This verifies reported domain membership and successful employee sign-in.

The [OU placement capture](../evidence/SUP-001-04-workstations-ou.png) shows SUPPORT01 and CLIENT01 in FischerLab Workstations.

The [user policy and IT access capture](../evidence/SUP-001-05-user-policy-it-read.png) shows successful computer and user policy refresh, then user RSoP for FISCHERLAB\amelia on SUPPORT01. FL-Users-DepartmentDrives is listed as applied; GG_IT_Users and DL_FS01_IT_Modify appear in the user's group list. Reading the IT restore-validation.txt file returns “FischerLab controlled file recovery test.” The captured user policy timestamp is October 13, 2026.

The [final validation capture](../evidence/SUP-001-06-trust-computer-policy-access-boundary.png) verifies the remaining checks:

- Elevated session identifies SUPPORT01; Test-ComputerSecureChannel returns True and reports the channel to corp.fischerlab.test is in good condition.
- Computer RSoP identifies SUPPORT01 in FischerLab Workstations and lists FL-Workstations-Baseline, FL-Workstations-BrowserSecurity, and Default Domain Policy as applied.
- Amelia's ordinary session shows the I: mapping to the IT share with status OK.
- Listing the Accounting share returns Access is denied with PermissionDenied / UnauthorizedAccessException. This is the expected department authorization boundary.

The provisioning acceptance checks are complete. DHCP lease provenance and exact Windows edition have not been independently captured; the address configuration, reported domain membership, and successful domain operations are documented. GPO application is verified, but each individual policy setting has not been separately tested.

## Issues and resolution

The initial computer name was changed to SUPPORT01. No failed provisioning checks were reported in the supplied evidence. The Accounting denial is expected behavior, not an unresolved incident.

## Verification and evidence

[Initial client addressing](../evidence/SUP-001-01-initial-addressing.png) · [Rename and connectivity verification](../evidence/SUP-001-02-name-dns-smb-port.png).

The domain, policy, trust, and department-access checks are supported by the linked captures; edition and address-lease evidence limitations are recorded above. The final validation screenshot documents domain trust, computer policy application, the IT drive mapping, and the Accounting denial.

## User update

SUPPORT01 is ready for the IT employee. Domain sign-in, workstation policies, the IT drive, and IT file access were verified. Accounting access remains restricted as intended. The workstation can now be used for the separate troubleshooting exercises.

