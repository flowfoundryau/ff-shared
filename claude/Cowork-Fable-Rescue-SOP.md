# SOP: Rescue handoffs from Fable-stuck Cowork sessions

*Created 2026-06-15 by Opus for David Jackson, Flow Foundry Consulting [linkedin.com/in/djackson](https://linkedin.com/in/djackson)*



**Claude Cowork error message:**

```text
Failed to authenticate. API Error: 403 Claude Fable 5 is not available. Please use Opus 4.8.
Learn more: https://www.anthropic.com/news/fable-mythos-access
```

---

## What this is

Cowork sessions created on **Claude Fable 5** are dead: Fable is retired, so they throw `403 Claude Fable 5 is not available`. The model is pinned **server-side**, so editing local files doesn't fix it (the app re-syncs the old value on launch). This SOP doesn't revive those sessions. It **rescues the work** out of them, one handoff per dead session, so you can resume in a fresh Opus 4.8 session.

It runs in **two phases**:

- **Phase 1 (once):** back up your Cowork sessions and turn every Fable session into a written handoff.
- **Phase 2 (repeat, per session, whenever you like):** open one handoff and resume that project.

The actors throughout: **🧑 you** and **🤖 Claude** (a fresh Opus 4.8 session).

---

## Setup

Open a new **Opus 4.8** Cowork session and give it this SOP (either paste this file's URL and ask Claude to read it, or download the `.md` and attach it to the session). Connect your **Documents** folder. Then do Phase 1.

Two ground rules, so nothing surprises you:

- **Closing Claude Desktop is optional.** A backup taken with the app open is fine (a daily open-app backup is a normal habit). Close it only if you want to be extra safe.
- **Only run code you can read.** The backup below is two short, single-line commands you can read in full. The SOP deliberately avoids long, opaque installer scripts. If any step ever looks too long, ask Claude to explain it line by line before you run it.

---

## Phase 1 (once): back up, then build the handoffs

**🧑 Step 1. Get 7-Zip** if you don't have it (Explorer's built-in zip fails on Cowork's paths, which run over 260 characters). 

Install from <https://www.7-zip.org>, or run (in Powershell):

```powershell
winget install --id 7zip.7zip -e
```

**🧑 Step 2. Back up your session store** to `Documents\cowork-rescue.zip` (credential files excluded). Run the block matching how Claude was installed:

Microsoft Store / WinGet / enterprise (MSIX) build:

```powershell
& "C:\Program Files\7-Zip\7z.exe" a -tzip "$env:USERPROFILE\Documents\cowork-rescue.zip" `
  "$env:LOCALAPPDATA\Packages\Claude_*\LocalCache\Roaming\Claude\local-agent-mode-sessions\*" `
  "-xr!.credentials.json"
```

Direct download (.exe) build:

```powershell
& "C:\Program Files\7-Zip\7z.exe" a -tzip "$env:USERPROFILE\Documents\cowork-rescue.zip" `
  "$env:APPDATA\Claude\local-agent-mode-sessions\*" `
  "-xr!.credentials.json"
```

Not sure which build you have? Run the first; if it archives nothing, run the second. (Prefer Claude to tailor the command to your machine? Ask it, it can generate and explain the exact line before you run it.)

**🤖 Step 3. Have Claude inspect the archive and write the handoffs.** With `cowork-rescue.zip` sitting in your connected Documents folder, paste this:

> I have a 7-Zip archive of my Claude Cowork session store at **Documents\cowork-rescue.zip**. Some sessions are dead because they were pinned to Claude Fable 5 (now retired). Work only from the archive, don't touch my live app files.
>
> 1. Without fully extracting, list every session: read each top-level `local_<guid>.json` and report its **title**, **model** (the `.model` field), and **guid**. Give me the model distribution and flag every session whose model contains **"fable"**.
> 2. **Write one handoff file per Fable session** (not a combined summary), named `Handoff-<short-title-slug>.md`, saved to my connected folder. Build each from that session's own transcript: the `.jsonl` under `local_<guid>/.claude/projects/<encoded>/<uuid>.jsonl` (JSON-lines; `text` blocks in `user` turns are my messages, `assistant` turns hold replies plus `tool_use` actions). Each handoff must contain: **objective**; **what was done** (chronological, key decisions/rules); **current file versions**; **open/undecided items**; the **exact next task** the session died on; and a **"Files & folders needed to resume"** section.
> 3. Build that "Files & folders needed" section from the metadata json (`cwd`, `userSelectedFolders`, `userApprovedFileAccessPaths`, `fsDetectedFiles`, and any file paths in `initialMessage`) as a concrete list of the **actual folders and files** to open. The real working files usually live in my own folders, not the zip, so this list is the important part.
> 4. Show me links to all the handoff files.

**Result:** a folder of per-session handoffs. The zip holds your full chat history and file paths, so keep it private and delete it when you're done.

---

## Phase 2 (repeat, per session): resume one project

Do this whenever you want to pick a stranded project back up. Once per session, in its own sitting.

**🧑 Step 1.** Open a new Opus 4.8 session, ideally in the **same Cowork space** as the original session so its project memory auto-loads. Connect the folder holding the handoffs.

**🤖 Step 2.** Paste this, passing in, or naming the handoff you're resuming:

> I'm resuming one rescued Cowork session. Read **Handoff-<name>.md**. Then **request access to the folders and files listed in its "Files & folders needed to resume" section** (actually issue the access request, don't just list them). Once I approve, pick up the work from the **"exact next task"** in the handoff.

**🧑 Step 3.** Approve the access requests. Claude resumes the work.

Repeat for each session you care about.

---

## Caveats / honesty notes

- **Claude can't run the backup itself, but it can hand you the command.** Its sandboxed Linux shell can't reach your Windows files or run commands on the host, so Phase 1 Step 2 is yours to run. Claude can still generate and explain the exact PowerShell first.
- **You cannot un-stick the dead sessions** by any local edit: the model is server-pinned and re-synced on launch. This SOP rescues content; it doesn't revive the session. An Anthropic-side migration off Fable is the only thing that would actually move one.
- **Install channel varies, the path mostly doesn't.** `pzs8sxrjxfjjc` is Anthropic's publisher identity, the same on every machine for the official app. What varies is MSIX (Store/WinGet/enterprise) vs the direct `.exe` download, which is why Step 2 has two variants.
- **Real working files live outside the zip.** A session's spreadsheets and docs are in your own folders, not the archive. The handoff's access list points to them; confirm those paths still exist before relying on them.
- **`Compress-Archive` (PowerShell's built-in) often fails on these long paths**, which is why Step 1 uses 7-Zip.
