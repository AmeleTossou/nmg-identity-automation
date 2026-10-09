**Walkthrough video (6 min):** https://www.loom.com/share/e43e64617d6e4068a6b58deb514a6861

# NMG Identity Automation with Powershell

PowerShell tooling for identity lifecycle management, built for
Northstar Medical Group.

## The Problem

The Process to identify former emplyee account depended only on one employee manually emailing IT whenever someone quit.After she retired, notifications stopped for 102 days.

After a manual review, 23 old account taking 11 hours over 4 days was found. However, it could not identify unreported departures, contractor accounts, or service accounts. 

## The Approach

Rather than comparing directory records against payroll records, these
tools created in the project query the domain controller directly for the last authentication
date of every account. That value does not depend on paperwork being
filed correctly or names matching between systems

## Before you start

- Windows Server with the ActiveDirectory PowerShell module

      Import-Module ActiveDirectory

- Rights to modify user objects in the domain
- An authorising ticket number, in the form NMG-0000
- A Disabled Users OU at the root of the domain
- A writable reports folder. Create it if it does not exist:

      New-Item -Path "C:\Reports\Offboarding" -ItemType Directory -Force

## Tools

### Find-StaleAccounts.ps1

Identifies enabled accounts that have not authenticated within a given
number of days, including accounts that have never authenticated.
Exports a timestamped CSV plus a summary recording the exact query used.

    .\Find-StaleAccounts.ps1
    .\Find-StaleAccounts.ps1 -Days 30
    .\Find-StaleAccounts.ps1 -Days 180 -IncludeDisabled

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Days` | int | 90 | Days without authentication before an account is considered stale |
| `-ReportPath` | string | C:\Reports | Where reports are written |
| `-IncludeDisabled` | switch | off | Include disabled accounts in results |

**Output:** a timestamped CSV of findings, and a summary file recording
the question that produced them so the report can be reproduced.

### Offboard-NMGUser.ps1

Performs all five steps of SOP-IAM-001 against a single account.
Documents the account and its group memberships, verifies that
record on disk, disables the account, stamps the authorising
ticket, removes all group memberships, and moves the account to
the Disabled Users OU.

    .\Offboard-NMGUser.ps1 -Username "jdoe" -Ticket "NMG-0214" -WhatIf
    .\Offboard-NMGUser.ps1 -Username "jdoe" -Ticket "NMG-0214"

| Parameter | Type | Default | Description |
|---|---|---|---|
| `-Username` | string | required | SamAccountName of the account to offboard |
| `-Ticket` | string | required | Authorising ticket, in the form NMG-0000 |
| `-ReportPath` | string | C:\Reports\Offboarding | Where evidence files are written |
| `-LogPath` | string | Logs\ | Where the run transcript is written |
| `-WhatIf` | switch | off | Runs every check and changes nothing |

**Output:** two timestamped CSV files per account recording what
it was and what it could reach, plus a transcript of the run.

### Get-NMGOffboardingStatus.ps1

Reports how many accounts have been offboarded and quarantined,
how many are offboarded but not yet moved, and how many are still
waiting. Takes no parameters and makes no changes of any kind.

    .\Get-NMGOffboardingStatus.ps1

**Output:** three counts and two named lists, printed to the
console. Nothing is written to disk.

## When the script refuses

A refusal is the tool working correctly. It stops before making
any change at all, and tells you why.

| Message | What it means | What to do |
|---|---|---|
| no account named X | The username is wrong, or the account is already gone | Check the spelling in Active Directory |
| already disabled | Somebody has handled this one before you | Read the description field for the ticket number |
| looks like a service account | This is not a person | Find the owner. It needs a different procedure |
| ticket should look like NMG-0000 | The ticket format is wrong | Use the full four digit form |
| export file is empty | The record could not be written | Check the reports folder exists and is writable |
| Disabled Users OU not found | The destination is missing | Steps 1 to 4 completed. Move the account by hand |

## What an offboarding leaves behind

- Two timestamped CSV files per account in `Evidence/`. One
  records the account, one records every group it could reach.
- One transcript per run in `Logs/`, naming every membership
  removed.
- The authorising ticket number stamped on the account
  description in Active Directory.

The CSV of group memberships is the only record that will ever
exist of what an account could reach beforehand. Active Directory keeps no
history of a removed memberships.

## Known limitations

- Three accounts were offboarded before step 5 was implemented
  and were moved into the Disabled Users OU manually afterwards.
  Their logs do not record the move.
- Handles one account per run. Bulk processing is not built yet.
- The service account check matches on a name prefix and a
  department. An unusually named service account could get past it.
  
  
## Repository Structure

    Scripts/        PowerShell tools
    Documentation/  Runbooks and process documentation
    Evidence/       Sample output and verification screenshots
    Logs/           Execution logs

## Environment

Windows Server with Active Directory Domain Services.
Requires the ActiveDirectory PowerShell module.

## About

## Why each safeguard exists

Every safeguard below looks like unnecessary caution to somebody
who was not here when it was added. Each one is recorded with the
consequence of removing it, so that a future change is a decision
rather than an assumption.

**Documentation happens before removal.** Active Directory keeps
no history of a removed group membership. The CSV written in step
1 is the only record of what an account could reach, and it is
also the only thing a rollback can restore from. Remove this step
and a failed bulk run has nothing to roll back to.

**Validation is separate from action.** A check placed inside the
action loop runs at the same speed as the action, with nobody
watching. Separating them creates a point at which a person reads
a report and decides. Collapse the phases and that point
disappears, along with the only control that catches a row naming
a current employee.

**The report has three tiers.** A reviewer reads the first few
rows of a list carefully and the remainder progressively less so.
Presenting forty rows of equal weight spends the reviewer's
attention on rows that needed none, and a row requiring a decision
can be approved without being read. Flattening the report removes
the control without changing a single check.

**A person approves before anything runs.** Deciding that a named
individual should not be offboarded is a judgement rather than a
rule. On 13 August a row naming a current employee was correctly
formatted, correctly spelled and pointed at a valid account. No
automated check would have refused it. It was declined by a person
reading one line.

### Before removing any of the above

Read this section and the findings documents that produced it.
Each safeguard was added in response to a specific failure that
was observed rather than imagined.


Built during the TotalThreat 30-Day Challenge in a simulated
healthcare environment. Northstar Medical Group is fictional.

Author: Amele Tossou

