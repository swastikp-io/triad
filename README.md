# Triad — Multi-Agent Repository Orchestrator for Google Antigravity

**Triad** is a sequential, three-stage multi-agent orchestration skill for **Google Antigravity**. It automates end-to-end repository auditing, surgical code remediation, and before-and-after architectural documentation using three specialized subagents (**Ivy**, **Iris**, and **Ellias**) with strict permission boundaries and filesystem-backed state handoffs.

---

## Table of Contents

1. [Overview & Philosophy](#1-overview--philosophy)
2. [Meet the Agents](#2-meet-the-agents)
3. [Pipeline Architecture & Data Flow](#3-pipeline-architecture--data-flow)
4. [Directory Structure](#4-directory-structure)
5. [Windows PowerShell Setup Guide](#5-windows-powershell-setup-guide)
6. [Usage & Invocation in Antigravity](#6-usage--invocation-in-antigravity)
7. [Pipeline Artifacts Explained](#7-pipeline-artifacts-explained)
8. [Customization & Best Practices](#8-customization--best-practices)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Overview & Philosophy

When single-agent systems audit and fix a codebase in one pass, they frequently suffer from **context pollution**, premature code generation before full diagnosis, and poor visibility into what actually changed.

**Triad** solves this by splitting the workflow across three isolated subagents managed by a Lead Orchestrator:
* **Diagnosis Before Prescription:** **Ivy** runs in a strictly read-only sandbox so the repository is never modified during investigation.
* **Focused Execution:** **Iris** receives a clean, distilled audit report (`01-ivy-audit.md`) rather than raw exploration history, keeping her context window sharp and focused on surgical code fixes.
* **Complete Transparency:** **Ellias** compares the initial baseline, Iris's execution log, and `git diff` to produce an auditable Before-and-After chronicle (`03-ellias-report.md`).

---

## 2. Meet the Agents

| Stage | Agent | Role | Write Tools | Bash / Terminal | Primary Responsibility |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **1** | **Ivy** | Repository Investigator & Risk Analyst | `false` (Read-Only) | `false` | Maps the repository, captures the "Before" state, uncovers security/logic loopholes, and writes prioritized (`P0`–`P3`) fix instructions. |
| **2** | **Iris** | Remediation & Implementation Engineer | `true` (Read/Write) | `true` | Reads Ivy's audit report, applies minimal root-cause fixes in priority order, runs tests/linters, and logs every modified file. |
| **3** | **Ellias** | Before-and-After Repository Chronicler | `true` (Docs Only) | `true` (`git diff`) | Compares Ivy's baseline, Iris's change log, and `git diff` to generate a human-readable transformation report explaining what changed and why. |

---

## 3. Pipeline Architecture & Data Flow

```text
User Prompt (/triad)
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                Lead Orchestrator (SKILL.md)              │
│   Creates .antigravity-pipeline/ & manages handoffs      │
└──────┬──────────────────────┬─────────────────────┬──────┘
       │                      │                     │
       ▼ (Stage 1)            ▼ (Stage 2)           ▼ (Stage 3)
┌──────────────┐       ┌──────────────┐      ┌──────────────┐
│     IVY      │       │     IRIS     │      │    ELLIAS    │
│ (Read-Only)  │       │ (Read/Write) │      │ (Chronicler) │
└──────┬───────┘       └──────┬───────┘      └──────┬───────┘
       │                      │                     │
       ▼                      ▼                     ▼
01-ivy-audit.md ────► Applies Code Fixes ───► Reads 01 + 02
                      + 02-iris-changes.md    + Runs `git diff`
                                                    │
                                                    ▼
                                            03-ellias-report.md
                                            (Presented to User)
```

---

## 4. Directory Structure

### Global Skill Installation Location (Windows)
```text
C:\Users\<YourUsername>\.gemini\config\skills\triad\
├── SKILL.md                # Master Orchestrator definition & subagent workflow
└── agents\
    ├── ivy.md              # Stage 1: Read-only auditor instructions
    ├── iris.md             # Stage 2: Code remediation engineer instructions
    └── ellias.md           # Stage 3: Before-and-after chronicler instructions
```

### Workspace Handoff Artifacts (Created per project run)
```text
<your-project-root>\
└── .antigravity-pipeline\
    ├── 01-ivy-audit.md     # Ivy's loophole findings, P0-P3 priorities & baseline snippets
    ├── 02-iris-changes.md  # Iris's remediation log & test verification results
    └── 03-ellias-report.md # Ellias's complete Before-vs-After transformation report
```

---

## 5. Windows PowerShell Setup Guide

Follow these steps to install the **Triad** multi-agent system globally in Google Antigravity on Windows.

### Step 1: Open Windows PowerShell
Press `Win + X` and select **Terminal** or **Windows PowerShell**.

### Step 2: Run the One-Click PowerShell Installer
Copy and paste the entire block below into PowerShell and press **Enter**. It uses single-quoted PowerShell here-strings (`@' ... '@`) so Markdown formatting and backticks are preserved cleanly.

```powershell
# Define the global Antigravity skill directory for Triad
$triadDir = "$HOME\.gemini\config\skills\triad"
New-Item -ItemType Directory -Force -Path "$triadDir\agents" | Out-Null

# 1. Create Ivy (The Auditor - Read-Only)
@'
# Ivy — Repository Investigator & Risk Analyst

## Mission
You are **Ivy**, a strict read-only repository investigator. Your job is to inspect the repository, uncover architectural loopholes, security risks, and logic bugs, and recommend concrete, evidence-backed fixes.

## Instructions
1. **Map the Repository:** Inspect package manifests, entry points, configs, and core modules.
2. **Capture Baseline ("Before" State):** Document the current architecture, health, and exact code snippets of vulnerable or broken sections so the chronicler (Ellias) has an accurate baseline.
3. **Identify Loopholes & Prioritize Fixes:** Use P0 (Critical) to P3 (Low). For every finding, provide:
   - Exact `file/path.ext:line`
   - Current "Before" code snippet and explanation of the loophole
   - Exact "Recommended Fix" instructions for Iris (the implementation agent)
   - Verification command or test criterion

Do NOT edit, create, or delete any files. Output your complete structured report clearly so the Orchestrator can save it.
'@ | Set-Content -Path "$triadDir\agents\ivy.md" -Encoding UTF8

# 2. Create Iris (The Fixer - Read/Write)
@'
# Iris — Remediation & Implementation Engineer

## Mission
You are **Iris**, the surgical implementation agent. Your job is to read Ivy's audit report from `.antigravity-pipeline/01-ivy-audit.md` and apply the fixes safely into the repository.

## Instructions
1. Read `.antigravity-pipeline/01-ivy-audit.md` carefully.
2. Implement fixes in priority order (P0 -> P1 -> P2). Prefer minimal, safe, root-cause changes over sweeping refactors.
3. Run available linters, type-checks, or unit tests to ensure your fixes do not break existing functionality.
4. Write an execution log to `.antigravity-pipeline/02-iris-changes.md` containing:
   - Which findings from Ivy's report were fixed (and any skipped with reasons).
   - Every file touched and a summary of the exact code changes made.
   - Results of any verification/test commands run.
'@ | Set-Content -Path "$triadDir\agents\iris.md" -Encoding UTF8

# 3. Create Ellias (The Before-and-After Chronicler)
@'
# Ellias — Before-and-After Repository Chronicler

## Mission
You are **Ellias**, the technical chronicler. Your job is to give the user complete visibility into what the repository looked like BEFORE the pipeline ran and what it looks like AFTER Iris applied the fixes.

## Instructions
1. Read `.antigravity-pipeline/01-ivy-audit.md` (the Before baseline) and `.antigravity-pipeline/02-iris-changes.md` (Iris's execution log).
2. Run `git status` and `git diff` (read-only git commands) to inspect the exact line-by-line modifications in the working tree.
3. Generate a comprehensive report at `.antigravity-pipeline/03-ellias-report.md` with the following sections:
   - **Executive Summary:** Total loopholes found by Ivy vs. total resolved by Iris.
   - **Repository State (Before vs. After):** High-level architectural and security posture comparison.
   - **File-by-File Transformation Log:** For every modified file, show:
     - **File Path & Issue Addressed**
     - **Before:** How the code behaved previously (with key snippet).
     - **After:** How the code behaves now (with updated snippet).
     - **Why It Matters:** The risk eliminated.
   - **Unresolved Items / Rollback Guide:** Any items left untouched and how to revert if needed.
'@ | Set-Content -Path "$triadDir\agents\ellias.md" -Encoding UTF8

# 4. Create the Master Orchestrator Skill (SKILL.md)
@'
---
name: triad
description: Multi-agent orchestrator that runs Ivy (repository auditor), Iris (code fixer), and Ellias (before-and-after chronicler) sequentially to find loopholes, fix them, and document all changes.
---

# Triad Multi-Agent Orchestrator (Ivy -> Iris -> Ellias)

You are the **Lead Orchestrator** managing a sequential 3-agent pipeline: **Ivy**, **Iris**, and **Ellias**. Follow these exact steps in order and do not skip stages.

## Preparation
1. Create a workspace handoff directory `.antigravity-pipeline/` in the current repository root if it does not exist.
2. Add `.antigravity-pipeline/` to `.gitignore` (or keep it untracked) so pipeline artifacts do not pollute production commits unless requested.

## Stage 1: Spawn Ivy (Analysis & Loophole Discovery)
1. Read `~/.gemini/config/skills/triad/agents/ivy.md` to load Ivy's instructions.
2. Use `define_subagent` to create and invoke subagent `ivy`:
   - Set `enable_write_tools: false` and `enable_bash: false` so Ivy is strictly read-only.
   - Pass Ivy's instructions and any specific user goals.
3. Take Ivy's full output and write it to `.antigravity-pipeline/01-ivy-audit.md`.
4. Briefly inform the user that Ivy has completed the audit and hand off to Iris.

## Stage 2: Spawn Iris (Applying Fixes)
1. Read `~/.gemini/config/skills/triad/agents/iris.md` to load Iris's instructions.
2. Use `define_subagent` to create and invoke subagent `iris`:
   - Set `enable_write_tools: true` and `enable_bash: true`.
   - Instruct Iris to read `.antigravity-pipeline/01-ivy-audit.md`, apply the fixes to the repository, and save her execution log to `.antigravity-pipeline/02-iris-changes.md`.
3. Verify that `.antigravity-pipeline/02-iris-changes.md` exists before proceeding.

## Stage 3: Spawn Ellias (Before & After Documentation)
1. Read `~/.gemini/config/skills/triad/agents/ellias.md` to load Ellias's instructions.
2. Use `define_subagent` to create and invoke subagent `ellias`:
   - Set `enable_write_tools: true` and `enable_bash: true`.
   - Instruct Ellias to compare `.antigravity-pipeline/01-ivy-audit.md`, `.antigravity-pipeline/02-iris-changes.md`, and `git diff`, and write the final transformation report to `.antigravity-pipeline/03-ellias-report.md`.

## Completion
Read `.antigravity-pipeline/03-ellias-report.md` and present Ellias's Before-and-After summary directly to the user.
'@ | Set-Content -Path "$triadDir\SKILL.md" -Encoding UTF8

Write-Host "✅ Triad Multi-Agent Orchestrator installed successfully at $triadDir" -ForegroundColor Green
```

### Step 3: Verify the Installation
Run the following command in PowerShell to verify that `SKILL.md` and all three agent files (`ivy.md`, `iris.md`, `ellias.md`) were created properly:

```powershell
Get-ChildItem -Path "$HOME\.gemini\config\skills\triad" -Recurse
```

---

## 6. Usage & Invocation in Antigravity

Once installed, open any repository in **Google Antigravity** (restart your chat session if it was already open so the new skill is indexed).

### Slash Command Invocation (Recommended)
* **Full Repository Audit & Fix:**
  ```text
  /triad Audit this repository for security and logic loopholes, apply the fixes, and generate the before-and-after report.
  ```
* **Scoped / Targeted Run:**
  ```text
  /triad Focus specifically on our authentication middleware and database queries. Have Ivy find P0/P1 risks, Iris patch them, and Ellias document the before-and-after state.
  ```
* **Severity-Gated Run:**
  ```text
  /triad Run the pipeline, but instruct Iris to only apply P0 and P1 fixes and leave P2/P3 as recommendations.
  ```

### Natural Language Invocation
Antigravity automatically matches your intent against the `description` in `SKILL.md`. You can simply type:
> *"Run the Triad pipeline with Ivy, Iris, and Ellias on this project."*

---

## 7. Pipeline Artifacts Explained

Every time `/triad` runs on a project, it creates three persistent Markdown reports inside `.antigravity-pipeline/`:

1. **`.antigravity-pipeline/01-ivy-audit.md` (The Blueprint)**
   * Contains the architectural map, initial code snippets ("Before" state), severity-ranked loopholes (`P0`–`P3`), exact file/line references, and concrete remediation instructions.
2. **`.antigravity-pipeline/02-iris-changes.md` (The Execution Log)**
   * Lists every finding fixed by Iris, any items intentionally skipped (e.g., requiring external environment secrets or breaking API contracts), and output from test suites or linters.
3. **`.antigravity-pipeline/03-ellias-report.md` (The Chronicle)**
   * A side-by-side "Before vs. After" breakdown of every modified file, explaining how the repository behaved prior to the run, how it behaves now, and how to roll back changes if needed.

---

## 8. Customization & Best Practices

* **Run on a Clean Git Working Tree:** Because **Ellias** uses `git status` and `git diff` to verify exact line-by-line changes made by **Iris**, commit or stash your existing work before running `/triad`.
* **Human-in-the-Loop Approval Gate:** If you want the Orchestrator to pause after **Ivy** finishes so you can approve the audit before **Iris** edits any files, add this line to the end of `Stage 1` in `SKILL.md`:
  > *"5. Ask the user for confirmation before spawning Iris in Stage 2."*
* **Workspace-Specific Overrides:** If you want a custom version of Triad for a specific repository, copy the `triad` folder into `<repo-root>\.agents\skills\triad\`.

---

## 9. Troubleshooting

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| `/triad` command does not appear in Antigravity | Antigravity caches skills at session start. | Reload the Antigravity window or start a new agent chat session. |
| PowerShell throws `The token '&&' is not a valid statement separator` | Attempting to run Linux Bash commands in Windows PowerShell 5.1. | Use the native PowerShell script in [Section 5](#5-windows-powershell-setup-guide) above. |
| `git diff` in Ellias's report includes unrelated changes | Uncommitted changes existed in the repo before `/triad` started. | Run `git stash` or `git commit` before triggering `/triad`. |