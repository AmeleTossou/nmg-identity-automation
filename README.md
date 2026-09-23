# nmg-identity-automation
Identity lifecycle automation with Powershell built during the TotalThreat 30-Day Challenge
# NMG Identity Automation with Powershell

PowerShell tooling for identity lifecycle management, built for
Northstar Medical Group.

## The Problem
The Process to identify former emplyee account depended only on one employee manually emailing IT whenever someone quit.After she retired, notifications stopped for 102 days.

After amanuel review, 23 old acount taking 11 hours over 4 days was found.However, it could not identify unreported deprtures, contractor accounts, or service accounts. 

## The Approach

Rather than comparing directory records against payroll records, these
tools created in the project query the domain controller directly for the last authentication
date of every account. That value does not depend on paperwork being
filed correctly or names matching between systems

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

## Repository Structure

    Scripts/        PowerShell tools
    Documentation/  Runbooks and process documentation
    Evidence/       Sample output and verification screenshots
    Logs/           Execution logs

## Environment

Windows Server with Active Directory Domain Services.
Requires the ActiveDirectory PowerShell module.

## About

Built during the TotalThreat 30-Day Challenge in a simulated
healthcare environment. Northstar Medical Group is fictional.

Author: Amele Tossou

