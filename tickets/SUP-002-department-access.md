# SUP-002 Troubleshoot Department File Access

**Status:** Resolved — departmental access restored and refreshed group token / drive mapping verified on October 13, 2026

## Simulated report

A new IT employee can sign in but cannot access the IT department share.

## Exercise boundary

Use a newly created synthetic test account for this exercise. Do not remove Amelia or other baseline users from their existing groups, and do not weaken share or NTFS permissions to make the test pass.

## Investigation plan

1. On DC01, create a dedicated account such as support.it.test in the existing IT user OU, with a password entered interactively. Initially leave it without GG_IT_Users membership.
2. On SUPPORT01, sign in as that account and record `whoami`, `whoami /groups`, DNS resolution, FS01 TCP 445 connectivity, and the exact IT file-access error.
3. Inspect the account's group memberships on DC01. Trace the existing GG_IT_Users membership into the domain-local resource group and inspect the share and NTFS permissions on FS01. Capture the actual configuration rather than assuming it matches the old baseline.
4. If the account is missing its required departmental group, add only the appropriate membership. Record why this is the smallest relevant change.
5. Sign out of Windows and sign back in as the test user to obtain a new logon token, then repeat the access check. Do not rely on a policy refresh alone to refresh group membership in the user's token.
6. Confirm the user can access the IT validation file and still encounters the expected Accounting boundary.

If the recorded failure has another cause, diagnose and document that cause instead. The setup is a simulation; the investigation results must come from the live lab.

## Actual symptom and user impact

The new synthetic account support.it.test cannot read the IT validation file. The [directory capture](../evidence/SUP-002-01-test-account-it-ou.png) shows Support IT Test in FischerLab Users IT. This OU placement alone does not establish group membership or grant share access.

The [failure capture](../evidence/SUP-002-02-connectivity-pass-access-denied.png) shows the session identity FISCHERLAB\support.it.test. FS01 resolves to 10.10.10.20, and TCP 445 succeeds from 10.10.20.101. Reading the existing IT restore-validation.txt file returns Access is denied with PermissionDenied / UnauthorizedAccessException.

## Investigation and root cause

The captured token does not list GG_IT_Users or DL_FS01_IT_Modify. Missing departmental membership is the leading explanation for the simulated failure. DNS resolution and TCP reachability succeed, so those checks do not support a DNS or blocked-port cause for this attempt.

The [directory membership capture](../evidence/SUP-002-03-directory-membership-chain.png) shows support.it.test belongs only to Domain Users. DL_FS01_IT_Modify contains GG_IT_Users.

The [permission capture](../evidence/SUP-002-04-share-ntfs-permissions.png) shows the IT share allows FISCHERLAB\DL_FS01_IT_Modify Change and BUILTIN\Administrators Full. The share folder grants the resource group NTFS Modify and Synchronize, and SYSTEM and Administrators FullControl. The displayed folder entries are explicit Allow entries.

These observations support missing GG_IT_Users membership as the cause of the intended simulated access failure. Folder-level permissions were inspected; the individual file's ACL was not separately queried. The subsequent captures document the membership correction and recovery checks.

## Change and validation

Requested correction: add support.it.test to GG_IT_Users only, preserving the existing resource-group nesting, share permissions, and NTFS permissions.

The [after-change access capture](../evidence/SUP-002-05-it-read-accounting-denied-token-mapping-pending.png) shows FISCHERLAB\support.it.test successfully reading the IT validation file while Accounting listing is still denied. Computer and user policy refresh report success.

However, the filtered whoami group query returns no matching rows, and Get-SmbMapping returns no mappings. This capture supports restored UNC file access and the Accounting boundary, but does not demonstrate a refreshed Windows token or an IT drive mapping. At this intermediate stage, directory membership after the change had not yet been captured. A stale logon token was a possible explanation, not an established cause. The subsequent captures establish directory membership and the refreshed token and mapping.

## Evidence and user update

The [directory correction capture](../evidence/SUP-002-06-corrected-directory-membership.png) shows support.it.test in Domain Users and GG_IT_Users. The [fresh-session validation capture](../evidence/SUP-002-07-refreshed-token-it-mapping.png) shows FISCHERLAB\support.it.test with GG_IT_Users and DL_FS01_IT_Modify enabled in its token. User and computer policy refresh succeed, and I: maps to the IT share with status OK.

The earlier after-change capture establishes the successful original UNC file read and continued Accounting denial. The fresh-session capture establishes the previously missing token and drive-mapping evidence. The read and Accounting denial were not repeated in this final capture; each result is attributed to its actual screenshot.

Root cause: the simulated onboarding omitted GG_IT_Users membership. Adding the departmental membership restored IT file access; a fresh sign-in then showed the expected Windows group token and drive mapping. A policy refresh in the earlier session had not demonstrated a refreshed token. No share or NTFS permission changes were required by the correction; no such changes were reported.

User update: IT file access and the I: drive are available. Accounting access remains restricted. The ticket is resolved based on the combined before/after evidence.

