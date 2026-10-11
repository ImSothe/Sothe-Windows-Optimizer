<div align="center">

# Sothe Windows Optimizer

**A Windows tuning tool for power users: no hidden dangerous settings, 100% reversible.**

[![Latest release](https://img.shields.io/github/v/release/ImSothe/Sothe-Windows-Optimizer?label=release&color=8b5cf6)](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/ImSothe/Sothe-Windows-Optimizer/total?color=22d3ee)](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases)
![Windows 10 | 11](https://img.shields.io/badge/Windows-10%20%7C%2011%20x64-0078D4?logo=windows&logoColor=white)
![.NET 8](https://img.shields.io/badge/.NET-8%20(bundled)-512BD4?logo=dotnet&logoColor=white)
![WPF](https://img.shields.io/badge/UI-WPF-5C2D91)
![Tests](https://img.shields.io/badge/tests-212%20passing-success)
![Signed updates](https://img.shields.io/badge/auto--update-signed%20ECDSA%20P--256-8b5cf6)
![License](https://img.shields.io/badge/license-freeware%20(closed%20source)-lightgrey)

[**⬇ Download the latest release**](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest) · [How it works](#architecture) · [Security](#security) · [Try it / contact](#want-to-try-it)

</div>

---

## What it is

Sothe Optimizer applies **70+ performance, latency, privacy and cleanup tweaks** for Windows, built for **competitive gaming** (CS2, Valorant, Overwatch…) and **cybersecurity labs** running VMs.

Unlike most "optimizer" apps, it **does not hide the dangerous settings**: it lets you touch Defender, the firewall, Spectre/Meltdown mitigations, Hyper-V or UAC. The one hard rule is that **everything must be undoable**: the previous value is saved before anything changes, and one panic button ("Revert ALL") restores it.

> **Honesty first.** What actually wins you a shooter is a high-refresh monitor, Reflex, stable frametimes, a wired connection and practice. Registry tweaks give you, at best, a single-digit percentage. This tool saves you hours of manual tuning — it doesn't work miracles.

### What's inside

| | |
|---|---|
| **Tweaks** | 70, across 13 categories (Performance, Input, Network, GPU, Power, Privacy, UI, Storage, Security, Debloat, Windows Update…) |
| **Risk levels** | `safe` 🟢 · `moderate` 🔵 · `high` 🟠 · `extreme` 🔴 (**FORBIDDEN**, hidden behind Advanced Mode) |
| **Apps (winget)** | 31 programs, batch install/uninstall |
| **Windows features** | 6 (Hyper-V, .NET 3.5, WSL…) |
| **Profiles** | Competitive · Cybersecurity · Clean · your own, with a shareable code (`SOTHE-1-…`) |
| **Repairs** | SFC, DISM, network reset, Windows Update repair, CHKDSK |

---

## Download & install

This repository **only ships the executable**. No installer, no dependencies — a single self-contained `.exe`.

1. Go to [**Releases**](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest) and download **`SotheOptimizer.exe`**.
2. *(Recommended)* Verify its integrity:

   ```powershell
   Get-FileHash .\SotheOptimizer.exe -Algorithm SHA256
   ```

   The result must match the `sha256` field in that release's `release.json`.
3. Double-click to run it. Windows will ask for **administrator rights** (UAC) — accept.
4. If SmartScreen shows up ("Windows protected your PC"): **More info → Run anyway**. The binary isn't Authenticode-signed yet (a code-signing certificate costs ~150–400 €/year); integrity is instead guaranteed by the signatures and hashes described below.
5. Read and accept the legal notice, **create a restore point** from inside the app, and start from a profile.

### Requirements

- **Windows 10 or 11, 64-bit.**
- **Administrator rights**: the app edits the registry (HKLM), services and system features. Without elevation Windows won't launch it; if you cancel the UAC prompt, the app simply doesn't open and nothing is touched.
- **No .NET install needed**: the runtime is bundled in the `.exe` (hence the ~160 MB size).
- Internet connection is **optional**: only used for licensing, updates and the winget catalog.

---

## Architecture

Sothe Optimizer is a **WPF app on .NET 8**, published as a single-file, self-contained executable. The logic is split into engine, data and UI:

```
┌─────────────────────────── UI (WPF) ──────────────────────────────┐
│ Optimizations · Apps (winget) · Repairs · Windows Features         │
└──────────────┬────────────────────────────────────┬────────────────┘
               │ apply / revert                     │ license / update
┌──────────────▼───────────────┐      ┌─────────────▼───────────────────────┐
│ Engine (SotheEngine)         │      │ Licensing (Keygen) · AppUpdater      │
│ registry · services · cmds   │      │ signed data OTA                      │
│ backups · reconciliation     │      └───────────────────────────────────────┘
└──────────────▲───────────────┘
               │ reads (JSON embedded in the exe, or a signed update)
        tweaks · apps · features · profiles
```

### Reversibility and reconciliation

- Every tweak is a list of **operations** (registry, service, command, Appx package) described in JSON — adding one never touches the code.
- Before applying, the **previous state** is saved to `%ProgramData%\SotheOptimizer\backups`.
- The main button **reconciles**: checked-but-not-applied gets applied; **unchecked-but-applied gets reverted**; already-applied is left alone. The system ends up exactly matching what's checked.
- A revert that **fails keeps its backup** (the tweak still counts as applied and can be retried) and is logged to `sothe.log`.
- There's a **dry-run simulation** to preview what would change without touching anything.

### Timer Resolution (native Windows API)

The "Fixed Timer Resolution" tweak asks the kernel for **0.5 ms** resolution via `NtSetTimerResolution` / `NtQueryTimerResolution` in `ntdll.dll` (P/Invoke). The platform timer is global: as long as *any* process holds the request, the whole system runs at that resolution. Because of that:

- A **scheduled task at logon** launches a lightweight resident process (`--timer-resident`, 0% CPU — it blocks on an event).
- A **global mutex** prevents duplicate instances.
- On exit (reverting the tweak, or logging off) the resident process **explicitly releases its request** (`NtSetTimerResolution(..., false)`) so resolution goes back to the system default and nothing drains extra battery. Revert signals a named event first and only force-kills as a last resort.
- The task runs elevated, so the `.exe` it launches is **copied into `Program Files`** (a folder only an administrator can write to) — the task never points at your Downloads folder.

### Signed auto-update (`AppUpdater`)

1. On startup it checks this repository's latest release (**HTTPS only**, GitHub hosts only).
2. It downloads `release.json` + `release.sig` and verifies the **ECDSA P-256** signature against the app's **embedded public key**. The private key lives only on the author's machine.
3. If the version is newer, it **streams** the `.exe` to disk, hashing it with **SHA-256** on the fly under a size cap; if the hash or size don't match what was signed, it's discarded and nothing is kept.
4. It installs when nothing else is in progress (or on the next launch): a script **waits for the process to exit, backs up the current `.exe`, swaps it in, relaunches**, and **rolls back** if anything fails.
5. No connection, GitHub down, or corrupt data: nothing happens. A failed update check is logged and the app keeps working.

Even if GitHub or your network were compromised, **without the author's private key no binary the app would actually run can be slipped in**.

### Licensing

- Validated with **[Keygen](https://keygen.sh)**: node-locked licenses (one PC per key) and 3-month trials.
- The **machine fingerprint** is a salted SHA-256 of the BIOS UUID and motherboard serial; never sent in the clear.
- The license is cached **DPAPI-encrypted** and re-validated in the background. There's a **14-day offline grace period** and detection of clocks that are behind or carry impossible dates.
- Free tweaks always work, and **reverting is always free**.

### Updatable data (OTA)

The tweak JSON files can be updated without reinstalling, but since they contain commands run as administrator, **only a signed manifest is accepted** — signed with the author's key, with every file's SHA-256 checked. An unsigned, hand-edited, or older JSON is ignored in favor of the copy embedded in the `.exe`.

---

## Security

Sothe Optimizer asks for administrator rights and runs commands, so it has been audited as such (manual static analysis plus 212 automated tests). Highlights:

| Surface | Mitigation |
|---|---|
| **Updater** | HTTPS + host allow-list · ECDSA P-256 signature · signed SHA-256 and size · streamed download with a size cap · backup-and-rollback swap |
| **Data folder** (`%ProgramData%\SotheOptimizer`) | Only trusts what an administrator or the system created; protected ACL (standard users get read-only); anything planted by someone else is removed |
| **Process launching** | `cmd.exe`, `powershell.exe` and `winget` are launched by **absolute path**, so an executable dropped next to the `.exe` can't run as administrator |
| **Commands** | winget / Windows Feature / service / Appx package names validated against strict patterns; `ProcessStartInfo.ArgumentList` instead of concatenated argument strings |
| **OTA data** | Per-file signature and hash; only a closed list of filenames is ever accepted |
| **Licensing** | DPAPI-encrypted cache, salted fingerprint, rejects a validation timestamp set in the future |

**Privacy.** No telemetry. The only network calls are: GitHub (update checks), Keygen (licensing: your key, a hashed/salted fingerprint and machine name) and winget/Microsoft (the app catalog) when you use it.

**Known limits.** A .NET program running on your own machine can never be fully protected from whoever runs it. This app is not a black box to someone willing to analyze it; the real protection for licensing lives server-side, in Keygen.

Found a vulnerability? Please open a [private security advisory](https://github.com/ImSothe/Sothe-Windows-Optimizer/security/advisories/new) (see [SECURITY.md](SECURITY.md)) instead of a public issue.

---

## Uninstall & undo

- Inside the app: **Revert ALL** restores every tweak from its backup.
- To remove the app, just delete `SotheOptimizer.exe`. If you enabled *Fixed Timer Resolution*, **revert that tweak first** (it removes the scheduled task and its copy in `Program Files`).
- Data lives in `%ProgramData%\SotheOptimizer` (backups, your own profiles, the log).

---

## Want to try it?

The source code is closed (see below), and the compiled build is obfuscated. If you'd like early/beta access, want to report something before it's public, or are a recruiter who wants a walkthrough: reach out on Instagram **[@SoySothe](https://www.instagram.com/soysothe)** or by email at **angelbazm@gmail.com**.

---

## License and ownership

**Sothe Windows Optimizer is proprietary freeware (closed source).** It is free to download and use for personal purposes; the source code is not distributed.

- **Owner and copyright holder:** Sothe ([@ImSothe](https://github.com/ImSothe)). Copyright © 2026 Sothe. All rights reserved.
- **What you may do:** download, install and use `SotheOptimizer.exe` on computers you own or administer, and share the link to this repository. Some features require a separate PRO license.
- **What you may not do:** modify, resell or redistribute the executable, present it as your own, or reverse engineer it (except where the law expressly allows it).
- **Full terms:** see [LICENSE](LICENSE), in English with the Spanish original included.

GitHub shows the license as "Other" (or "NOASSERTION") because this is a custom, proprietary license rather than an open-source one such as MIT. That is intentional.

## Legal notice

Sothe Optimizer modifies deep Windows settings. **Use it at your own risk**: some tweaks disable security protections or can cause instability. Always create a restore point before applying changes. The software is provided "AS IS", with no warranty. See [LICENSE](LICENSE).

## Source code & engineering notes

The source code is **not distributed** in this repository; only signed binary releases are published here (see [LICENSE](LICENSE)). For anyone evaluating the engineering rather than running the app:

- WPF / .NET 8, single-file self-contained publish.
- `ntdll` P/Invoke for 0.5 ms timer resolution, with an explicit release on exit instead of relying on process teardown.
- ECDSA P-256 **signed auto-updater**: HTTPS + host allow-list, streamed SHA-256 verification, a declared-size cap, and a backup-and-rollback swap script.
- **Keygen** node-locked licensing with a DPAPI-protected cache and a salted hardware fingerprint.
- Signed OTA data updates, hardened `%ProgramData%` storage (ownership + ACL checks), absolute-path process launching.
- The public build is **obfuscated** (Obfuscar) before packaging.
- **212 automated tests** (xUnit) covering the engine, the updater, licensing and the security hardening above.
