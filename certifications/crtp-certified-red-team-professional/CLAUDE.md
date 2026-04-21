# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo Nature — Read Before Acting

This repository is a **GitBook-backed git repo**. The content is authored in GitBook and synced here automatically.

- **NEVER run `git pull`, `git push`, or any git write command** unless explicitly asked. The user manages sync manually.
- Notes are **not 100% complete** — exercises are ongoing, some files are empty stubs.
- The source of truth for what each exercise requires is the official lab manual: `LabManual_files/LabManual.html`

## Purpose

Study notes and cheatsheets for the **CRTP (Certified Red Team Professional)** certification — an Active Directory red teaming lab course. The user uses this repo to:
- Work through lab exercises step by step
- Ask questions and resolve errors during labs
- Get help during the actual exam

## Repository Structure

```
/
├── indice.md                          # Master index — start here for navigation
├── CLAUDE.md                          # This file
├── README.md                          # Course overview (23 LOs, 59 tasks)
├── LabManual_files/
│   └── LabManual.html                 # Official CRTP lab manual (authoritative)
├── learning-objectives/
│   └── learning-objetive-{1-20}.md   # Personal notes per exercise (some incomplete)
├── class-notes/
│   ├── class-one-notes.md            # PowerShell tradecraft, InviShell, AMSI
│   └── class-two-notes.md            # .NET tradecraft, loaders, MDE bypass
└── cheatsheet-fast-commands-*.md     # Quick-reference attack phase cheatsheets
```

**Note on filenames**: There are typos in filenames that must be preserved exactly:
- `learning-objetive-*` (not "objective")
- `learning-objtetive-7.md` (double typo in LO-7)
- `cheetsheet-fast-commands-applocker-policy.md` (inside `cheatsheet-fast-commands-explotation/`)

## Lab Environment (CRTP)

The exercises target a fictional AD forest. Key machines and accounts seen in notes:
- **dcorp-dc** — Domain Controller of `dollarcorp.moneycorp.local`
- **dcorp-mgmt**, **dcorp-adminsrv**, **dcorp-ci** — member servers
- **svcadmin**, **websvc**, **appadmin** — service accounts used in Kerberoasting
- **studentX** — attacker account (student VM)
- Forest root: `moneycorp.local` / Child domain: `dollarcorp.moneycorp.local` / External: `eurocorp.local`

All tooling lives at `C:\AD\Tools\` on the student Windows VM.

## 23 Learning Objectives — Quick Map

| LO | Topic |
|----|-------|
| 1 | PowerShell bypass: InviShell, AMSI, Script Block Logging, CLM |
| 2 | ACL enumeration (Domain Admins group, PowerView) |
| 3 | InviShell setup + AD enumeration continuation |
| 4 | Forest trust enumeration (PowerView + ADModule) |
| 5 | Local PrivEsc: PowerUp, WinPEAS, PrivEscCheck, Jenkins abuse |
| 6 | GPO abuse for admin access on dcorp-ci |
| 7 | Kerberoasting + OverPass-the-Hash + AppLocker bypass |
| 8 | Golden Ticket (extract DC hashes, forge with Rubeus) |
| 9 | Silver Ticket (HTTP + WMI services) |
| 10 | Diamond Ticket |
| 11 | DSRM persistence |
| 12 | DCSync (replication rights abuse, krbtgt hash) |
| 13 | Security descriptor modification (WMI/PSRemoting without DA) |
| 14 | Kerberoasting SQL Server service account |
| 15 | Unconstrained Delegation + Printer Bug → Enterprise Admin |
| 16 | Constrained Delegation (users + computers → DCSync) |
| 17 | ACL Write permissions abuse on computer objects |
| 18 | Cross-forest escalation via trust key (DA → EA) |
| 19 | Cross-forest escalation via krbtgt hash (inter-realm TGT) |
| 20 | (stub — not yet documented) |
| 21 | AD CS attacks: ESC1, ESC3 (DA and EA escalation) |
| 22 | MDE/MDI evasion, full attack chain without alerts |
| 23 | Tools transfer, LSASS dump via custom APIs, ASR bypass, custom loader |

When the user asks about a specific LO, always check:
1. The personal notes file (`learning-objectives/learning-objetive-N.md`)
2. The official lab manual (`LabManual_files/LabManual.html`) for the authoritative steps

## Key Tools (on student VM at C:\AD\Tools\)

| Tool | Purpose |
|------|---------|
| `InviShell\RunWithRegistryNonAdmin.bat` | Start stealthy PS session (AMSI + logging bypass) |
| `PowerView.ps1` | Offensive AD enumeration |
| `ADModule-master\` | Microsoft AD module (legitimate, stealthy) |
| `Loader.exe` | In-memory .NET assembly execution |
| `Rubeus.exe` | Kerberos attacks (Kerberoast, tickets, delegation) |
| `SafetyKatz.exe` / `BetterSafetyKatz.exe` | Credential dumping (Mimikatz variant) |
| `PowerUp.ps1` | Local privilege escalation enumeration |
| `Find-PSRemotingLocalAdminAccess.ps1` | Lateral movement: find machines with local admin |
| `Invoke-SessionHunter` | Find DA sessions on machines |

## Working with the Lab Manual

The official lab manual (`LabManual_files/LabManual.html`) is the ground truth. When the user is stuck on a step, read the relevant section of the HTML to get the exact commands and expected output. Use `grep` on the HTML to find specific LO sections.
