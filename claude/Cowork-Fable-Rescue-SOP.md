# SOP — Rescue handoffs from Fable-stuck Cowork sessions

**Claude Cowork Error Message:** 
**Failed to authenticate. API Error: 403 Claude Fable 5 is not available. Please use Opus 4.8. Learn more: https://www.anthropic.com/news/fable-mythos-access**

**What this is for.** Claude Cowork sessions that were created on **Claude Fable 5** are now dead — Fable is retired, so the session throws `403 Claude Fable 5 is not available` and won't run. The model is pinned **server-side**, so editing local files does **not** fix it (the app re-syncs the old value on launch). What you *can* do is **extract the work** — transcripts, decisions, output files, and the folder/file access each session used — into a clean "handoff" so you can resume in a fresh Opus 4.8 session.

**Why the archive step.** A running Claude can't reliably read/modify its own live session store, and Windows Explorer's "Send to → Compressed folder" fails on the very long paths Cowork uses (>260 chars). 7-Zip handles long paths and gives you a clean, point-in-time snapshot. You then drop that archive somewhere normal (e.g. Documents) and let a working session read it.

---

## Step 1 — Close Cowork / the Claude app

Fully quit it (check the system tray) so nothing is mid-write when you snapshot.

## Step 2 — Find the session store

It's one of these (depends on how Claude was installed — both channels exist on Windows 11):

- **MSIX build** (Microsoft Store, WinGet, or enterprise MSIX):
  `%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\local-agent-mode-sessions\`
  The `pzs8sxrjxfjjc` is Anthropic's publisher-identity hash — it's the **same on every machine** for the official app, so this literal path is reliable. (Inside the MSIX sandbox the app *thinks* it writes to `%APPDATA%\Claude`; Windows redirects that here.)
- **Direct `.exe` install** (Win32 installer from claude.ai/download):
  `%APPDATA%\Claude\local-agent-mode-sessions\`

Quick way to tell which you have: if `%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\` exists, it's MSIX; otherwise look under `%APPDATA%\Claude\`.

Inside, drill through the two GUID levels (`<account>\<org>\`) until you reach the folder that directly contains lots of **`local_<guid>`** folders and matching **`local_<guid>.json`** files. That's the session store. Each session = one `local_<guid>/` folder + one `local_<guid>.json` metadata file beside it.

## Step 3 — Snapshot it with 7-Zip

Install 7-Zip if needed, then run **PowerShell** (not Explorer). Pick the line matching your install:

**Store/MSIX build:**

```powershell
& "C:\Program Files\7-Zip\7z.exe" a -tzip "$env:USERPROFILE\Documents\cowork-rescue.zip" `
  "$env:LOCALAPPDATA\Packages\Claude_*\LocalCache\Roaming\Claude\local-agent-mode-sessions\*" `
  "-xr!.credentials.json"
```

**Standard install:**

```powershell
& "C:\Program Files\7-Zip\7z.exe" a -tzip "$env:USERPROFILE\Documents\cowork-rescue.zip" `
  "$env:APPDATA\Claude\local-agent-mode-sessions\*" `
  "-xr!.credentials.json"
```

Notes:

- `-xr!.credentials.json` excludes the OAuth token files. **Do this** — the store otherwise contains live credentials.
- **Smaller archive** (metadata + transcripts only, skips big output/node_modules): add `-ir!local_*.json -ir!*.jsonl` and drop the `\*` so it reads: `"...\local-agent-mode-sessions" -ir!local_*.json -ir!*.jsonl -xr!.credentials.json`. That's all the handoff actually needs.
- **Treat the archive as confidential** — it holds full chat content and file paths. Only feed it to your own Claude session; don't email it around.

## Step 4 — Hand it to a fresh Opus session

Start a **new Cowork session on Opus 4.8**, in the **same Cowork space** as the dead sessions if you can (so existing project memory auto-loads). Connect the folder that holds `cowork-rescue.zip` (e.g. Documents), then paste the prompt below.

---

## Prompt to paste into the new Opus session

> I have a 7-Zip archive of my Claude Cowork session store at **Documents\cowork-rescue.zip**. Some sessions are dead because they were pinned to Claude Fable 5 (now retired). Do this, working only from the archive (don't touch my live app files):
>
> 1. Without fully extracting, list every session: read each top-level `local_<guid>.json` and report its **title**, **model** (the `.model` field), and **guid**. Give me the model distribution and flag every session whose model contains **"fable"**.
> 2. For each Fable session, build a **handoff** from its transcript. The transcript is the `.jsonl` under `local_<guid>/.claude/projects/<encoded>/<uuid>.jsonl` — it's JSON-lines with `{type, message:{role, content[]}}`; the `text` blocks in `user` turns are my messages, and `assistant` turns hold replies + `tool_use` actions. Summarise: objective, what was done (chronological, with key decisions/rules), current file versions, open/undecided items, and the **exact next task** the session died on.
> 3. For each, list the **access it needs to resume** — pull `cwd`, `userSelectedFolders`, `userApprovedFileAccessPaths`, `fsDetectedFiles`, and any file paths in `initialMessage` from the metadata json, and turn them into a clean list of folders/files to grant a new session.
> 4. Save each handoff as a markdown file in my connected folder and show me the links.
>
> Note: the working files for these projects usually live in my own folders (per `userSelectedFolders` / the vault path), **not** inside the archive — so the access list matters more than the files in the zip.

---

## Caveats / honesty notes

- **You cannot un-stick the dead sessions** by any local edit — the model is server-pinned and re-synced on launch. This SOP rescues content; it doesn't revive the session. If Anthropic later offers an in-app migration off Fable, that's the only thing that would actually move the session.
- The package hash `pzs8sxrjxfjjc` is **stable across machines** for the official app (it's the publisher identity, not a per-install value), so the literal path is dependable; the `Claude_*` wildcard is just a safety net for non-official repackages. What genuinely varies is **which channel** the company used — MSIX vs direct `.exe` — so check both base paths.
- A session's real working files (spreadsheets, docs) typically live in the user's own folders, which are **not** in this archive. The handoff's access list tells you where they are; confirm those paths still exist before relying on them.
- `Compress-Archive` (PowerShell's built-in) has historically failed on these long paths — that's why this SOP specifies 7-Zip.
