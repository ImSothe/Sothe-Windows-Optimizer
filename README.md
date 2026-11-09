> **This repository is archived.** The project continues as **Toji**, with new tweaks, signed releases and the current
> source: **[ImSothe/Toji-Windows-Optimizer](https://github.com/ImSothe/Toji-Windows-Optimizer)**. Please download and
> report issues there. Old releases (v1.1.0 to v1.3.0) stay downloadable here for reference; v1.4.0 and later are published only in the Toji repository.

<div align="center">

# Toji

**A Windows tuning tool for power users: no hidden dangerous settings, and every change can be put back.**

[![CI](https://github.com/ImSothe/Toji-Windows-Optimizer/actions/workflows/ci.yml/badge.svg)](https://github.com/ImSothe/Toji-Windows-Optimizer/actions/workflows/ci.yml)
[![Latest release](https://img.shields.io/github/v/release/ImSothe/Sothe-Windows-Optimizer?label=release&color=8b5cf6)](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest)
![Windows 10 | 11](https://img.shields.io/badge/Windows-10%20%7C%2011%20x64-0078D4?logo=windows&logoColor=white)
![.NET 8](https://img.shields.io/badge/.NET-8%20(bundled)-512BD4?logo=dotnet&logoColor=white)
![WPF](https://img.shields.io/badge/UI-WPF-5C2D91)
![Tests](https://img.shields.io/badge/tests-172%20passing-success)
![Coverage](https://img.shields.io/badge/line%20coverage-87.5%25%20(non--UI)-success)
![Signed updates](https://img.shields.io/badge/auto--update-signed%20ECDSA%20P--256-8b5cf6)
![License](https://img.shields.io/badge/license-MIT-blue)

[**⬇ Download**](#download--install) · [How it works](#architecture) · [Security](#security) · [Build](#build-from-source) · [Known limits](#known-limits)

<img src="docs/images/toji-optimizations.png" width="860" alt="Toji: tweaks list with risk levels, profiles, search and the Apply / Revert all bar" />

</div>

---

> **This repository hosts the signed release builds that Toji's auto-update downloads. The source code, issues and development live in [Toji-Windows-Optimizer](https://github.com/ImSothe/Toji-Windows-Optimizer).**


## What it is

Toji applies **70 performance, latency, privacy and cleanup tweaks** to Windows, built for **competitive gaming**
(CS2, Valorant, Overwatch...) and **cybersecurity labs** running VMs.

Unlike most "optimizer" apps it **does not hide the dangerous settings**: it lets you touch Defender, the firewall,
Spectre/Meltdown mitigations, Hyper-V or UAC, behind an explicit *Advanced Mode* and a confirmation. The one hard rule
is that **what it changes, it can put back**: the previous state is saved *before* anything changes, and one panic
button ("Revert ALL") restores it in the right order.

> **Honesty first.** What actually wins you a shooter is a high-refresh monitor, stable frametimes, a wired connection
> and practice. Registry tweaks give, at best, a single-digit percentage, and some tweaks here have **no measured
> effect at all** (see [Known limits](#known-limits)). This tool saves you hours of manual tuning; it does not work
> miracles.

<table>
<tr>
<td><img src="docs/images/toji-apps.png" alt="Apps tab (winget)" /></td>
<td><img src="docs/images/toji-repairs.png" alt="Repairs tab" /></td>
<td><img src="docs/images/toji-features.png" alt="Windows features tab" /></td>
</tr>
</table>

### What's inside

| | |
|---|---|
| **Tweaks** | 70, across 13 categories (Performance, Input, Network, GPU, Power, Privacy, UI, Storage, Security, Debloat, Windows Update...), each described as data in `tweaks.json` |
| **Risk levels** | `safe` 🟢 · `moderate` 🔵 · `high` 🟠 · `extreme` 🔴 (**FORBIDDEN**, hidden behind Advanced Mode) |
| **Apps (winget)** | 31 programs, batch install/uninstall |
| **Windows features** | 6 (Hyper-V, .NET 3.5, WSL...) |
| **Profiles** | Competitive · Cybersecurity · Clean · your own, with a shareable code (`SOTHE-1-...`) |
| **Repairs** | SFC, DISM, network reset, Windows Update repair, CHKDSK |

> The UI is in **Spanish** and so are most code comments; the documentation is in English. Internationalising the UI is
> on the roadmap.

---

## Download & install

Signed binary releases are published in
[**Sothe-Windows-Optimizer › Releases**](https://github.com/ImSothe/Sothe-Windows-Optimizer/releases/latest), the
repository existing installs poll for updates. A single self-contained `.exe`: no installer, no dependencies.

> **Release status.** `v1.2.0` is the first open-source build (MIT, no licensing). The update-signing key was replaced, so installs of `v1.1.0` or older (the closed-source builds) will not auto-update: download `v1.2.0` once by hand; from there auto-update works again.

1. Download **`SotheOptimizer.exe`** (the file keeps its original name so existing installs update in place).
2. *(Recommended)* verify it: `Get-FileHash .\SotheOptimizer.exe -Algorithm SHA256` must match the `sha256` field of
   that release's `release.json`.
3. Double-click it. Windows asks for **administrator rights** (UAC): accept.
4. If SmartScreen shows "Windows protected your PC": **More info → Run anyway**. The binary is not Authenticode-signed
   yet (a code-signing certificate costs ~150-400 €/year); update integrity relies on the signatures described below.
5. Read and accept the legal notice and start from a profile. A **restore point is created automatically** before your
   first batch of changes (you can turn that off with the checkbox next to the buttons).

### Requirements

- **Windows 10 or 11, 64-bit.**
- **Administrator rights**: the app edits the registry (HKLM), services and system features. If you cancel the UAC
  prompt the app simply does not open and nothing is touched.
- **No .NET install needed**: the runtime is bundled in the `.exe` (hence the ~155 MB size).
- An Internet connection is **optional**: only used for update checks and the winget catalog.

## Build from source

```powershell
dotnet build SotheOptimizer.sln -c Release
dotnet test tests/SotheOptimizer.Tests -c Release
dotnet publish src/SotheOptimizer -c Release      # one self-contained SotheOptimizer.exe, runtime included
```

Requires the .NET 8 SDK on Windows 10/11 x64. `tools/OtaSigner` generates the signing key pair and signs releases and
data files; **the private key is never in the repo**. The (Spanish) build guide is
[src/SotheOptimizer/BUILD.md](https://github.com/ImSothe/Toji-Windows-Optimizer/blob/main/src/SotheOptimizer/BUILD.md).

---

## Architecture

Toji is a **WPF app on .NET 8**, published as a single-file, self-contained executable. The logic is split into
engine, data and UI:

```
┌─────────────────────────── UI (WPF) ──────────────────────────────┐
│ Optimizations · Apps (winget) · Repairs · Windows Features         │
└──────────────┬────────────────────────────────────┬────────────────┘
               │ apply / revert                     │ update
┌──────────────▼───────────────┐      ┌─────────────▼───────────────────────┐
│ Engine (SotheEngine)         │      │ AppUpdater (signed self-update)      │
│ registry · services · cmds   │      │ DataUpdater (signed data, OTA)       │
│ write-ahead backups · LIFO   │      └───────────────────────────────────────┘
└──────────────▲───────────────┘
               │ reads (JSON embedded in the exe, or a signed update)
        tweaks · apps · features · profiles
```

### Reversibility and reconciliation

- Every tweak is a list of **operations** (registry, per-subkey registry, service, command, Appx package) described in
  JSON, so adding one never touches the code.
- The main button **reconciles**: checked-but-not-applied gets applied; **unchecked-but-applied gets reverted**;
  already-applied is left alone. The system ends up matching what is checked. There is also a **dry-run** that shows
  what would change without touching anything.
- **Backups are written before every operation**, atomically, in `%ProgramData%\SotheOptimizer\backups`, with a
  sequence number. A crash mid-apply still leaves a way back, and re-applying an applied tweak is a no-op (it would
  otherwise overwrite the original state with the modified one).
- **Reverting is LIFO**: the last tweak applied is the first one undone, and inside a tweak the last operation is
  undone first. This matters because two tweaks can write the same key (`defender_full_off` and
  `defender_realtime_off` both write `DisableRealtimeMonitoring`): the second one's "previous value" is whatever the
  first left, so undoing them in the wrong order leaves Defender's real-time protection **off** after "Revert all".
  An earlier version had exactly that bug; it is covered by regression tests against a sandbox registry key.
- **Command tweaks restore the real previous state, not a default.** A command op can declare a read-only `capture`
  script (power plan, `bcdedit` values, hibernation, scheduled tasks, MSI...). Its output is stored in the backup and
  handed to `apply` and `revert` as `$captured` (as a quoted string, never pasted as code). If the capture fails,
  nothing is applied. Disabling a service stops it; reverting restores its start type **and** starts it again if it was
  running.
- A revert that **fails keeps its backup** (the tweak still counts as applied and can be retried) and is logged.

### Verified on a real machine, not just in unit tests

`tools/RoundTripCheck` applies and reverts tweaks on the actual Windows it runs on and diffs a snapshot of the system
(registry values, power plans and their values, services and their start type, scheduled tasks, hibernation, `bcdedit`,
HPET, MSI) before and after. On a Windows 11 machine that was already heavily tuned:

- **47 of 47** tweaks (all `safe` and `moderate` ones plus `hpet_off`, `bcd_timer` and `gpu_msi_mode`) restored the
  exact previous state after apply → revert.
- Applying all 47 and then reverting newest-first ("Revert all") left **zero differences** from the starting state.
- The resident timer process holds the calibrated resolution, refuses a second instance and releases it on stop.

The first run **found real bugs** that unit tests could not: reverts restored hard-coded defaults, so a custom power
plan was replaced, an existing "Ultimate Performance" plan deleted, hibernation turned back on and existing `TCPNoDelay`
values removed. Those are fixed (see the commit history) and covered by tests.

Honest limits of that run: 36 of the 47 tweaks were already applied on that machine, so they showed no *change* (the
restore check still holds); `firewall_off`, `hypervisor_off`, the Defender/UAC/VBS family, `reserved_storage_off` and
the Appx debloat were **not** run live (the tool refuses security-weakening tweaks by default).

### Timer Resolution, measured

The "Fixed Timer Resolution" tweak asks the kernel for a finer system timer through `NtSetTimerResolution` /
`NtQueryTimerResolution` in `ntdll.dll` (P/Invoke). Asking for exactly 0.5 ms is not always what makes `Sleep(1)` most
accurate, so it **calibrates** at apply time. Measured on one Windows 11 machine (supported range 0.500-15.625 ms), 4
alternating rounds of 150 samples:

| Requested | Mean `Sleep(1)` |
|---|---|
| default timer | 15.58 ms |
| 0.500 ms | 1.54 ms |
| 0.515 ms | **1.16 ms** |

It costs a little more jitter (stddev ~0.22 ms vs ~0.16 ms) and is hardware dependent, hence the measured sweep.

- A **scheduled task at logon** launches a lightweight resident process (`--timer-resident --resolution N`, ~0 % CPU:
  it blocks on an event) with the calibrated value. A **global mutex** prevents duplicates.
- On exit it **explicitly releases** its request so resolution returns to the system default. Revert signals a named
  event first and only force-kills as a last resort.
- The task runs elevated, so the `.exe` it launches is **copied into `Program Files`** (a folder only an administrator
  can write to), never pointed at Downloads.
- A measured negative result: on Windows 11 a resident process drops the *system* clock, but **other** processes'
  `Sleep(1)` stays at ~15.5 ms unless the `GlobalTimerResolutionRequests` opt-in is set (the tweak does it; it may
  need a reboot). The standalone library is [WindowsTimerResolution](https://github.com/ImSothe/WindowsTimerResolution).

### Signed auto-update (`AppUpdater`)

1. On startup it checks the latest release of this project (**HTTPS only**, GitHub hosts only).
2. It downloads `release.json` + `release.sig` and verifies the **ECDSA P-256** signature against the app's **embedded
   public key**. The private key lives only on the author's machine.
3. If the version is newer, it **streams** the `.exe` to disk, hashing it with **SHA-256** on the fly under a size cap;
   if the hash or size do not match what was signed, it is discarded and nothing is kept.
4. It installs when nothing else is in progress (or on the next launch): a script **waits for the process to exit,
   backs up the current `.exe`, swaps it in, relaunches**, and **rolls back** if anything fails.
5. No connection, GitHub down, or corrupt data: nothing happens, the check is logged and the app keeps working.

Even if GitHub or your network were compromised, **without the author's private key no binary the app would actually
run can be slipped in**. A standalone version of the same idea lives in
[SignedUpdater](https://github.com/ImSothe/SignedUpdater).

### Updatable data (OTA)

The tweak JSON files can be updated without reinstalling, but since they contain commands run as administrator, **only
a signed manifest is accepted** (author's key, SHA-256 of every file checked). An unsigned, hand-edited or older JSON is
ignored in favour of the copy embedded in the `.exe`. The mechanism is implemented and tested but **currently
disabled** (no base URL configured).

---

## Security

Toji asks for administrator rights and runs commands, so it was audited as such (manual review plus automated
tests). Highlights:

| Surface | Mitigation |
|---|---|
| **Updater** | HTTPS + host allow-list · ECDSA P-256 signature · signed SHA-256 and size · streamed download with a size cap · backup-and-rollback swap |
| **Data folder** (`%ProgramData%\SotheOptimizer`) | Only trusts what an administrator or the system created; protected ACL (standard users get read-only); anything planted by someone else is removed |
| **Process launching** | `cmd.exe` and `powershell.exe` are launched by **absolute path**, so an executable dropped next to the `.exe` cannot run as administrator |
| **Commands** | winget / Windows Feature / service / Appx names validated against strict patterns; `ArgumentList` instead of concatenated arguments; captured state is passed as a quoted string, never as code |
| **OTA data** | Per-file signature and hash; only a closed list of filenames is ever accepted |
| **Share codes** | Importing a `SOTHE-1-...` code only ticks boxes that already exist; nothing runs until you press Apply |

**Privacy.** No telemetry, no accounts, no licensing. The only network calls are GitHub (update checks) and
winget/Microsoft (the app catalog) when you use the Apps tab.

**Known limits.** A .NET program running on your own machine cannot be fully protected from whoever runs it, and an
elevated program is a juicy target: please read [SECURITY.md](https://github.com/ImSothe/Toji-Windows-Optimizer/blob/main/SECURITY.md) and report vulnerabilities privately.

---

## Tests

```powershell
dotnet test tests/SotheOptimizer.Tests -c Release
```

**172 tests**, ~2 s. Line coverage is **87.5 %** (788 / 901) and branch coverage 72.5 %, measured with `coverlet` over
the engine, updaters, signing, storage hardening and timer code. The WPF windows (about 1.6k lines) are **not**
covered, so that figure is not the coverage of the whole product. Registry round-trip tests only ever touch
`HKCU\Software\TojiTests\<guid>`.

## Known limits

- **Not every tweak is reversible.** Appx debloat cannot be reinstalled automatically (the app says so). A setting that
  was never explicitly set in a power plan (for example `CPMINCORES` on some custom plans) has no value to put back, so
  its revert is skipped. `reserved_storage_off` and the security-weakening tweaks have not been verified live.
- **No measurements for most tweaks.** Several performance tweaks (for example `nagle_off`, `gaming_gpu_priority`,
  `game_mode`) are community folklore with no measured effect on modern Windows; treat them as optional. On current
  Windows, Defender tweaks are also ignored while Tamper Protection is on.
- `HKCU` tweaks apply to the account that elevated the process, which is not the interactive user if a different
  administrator's credentials were used at the UAC prompt.
- The binary is not Authenticode-signed, so SmartScreen warns on first run. Update integrity does not depend on it.

## Uninstall & undo

- Inside the app: **Revert ALL** restores every tweak from its backup, newest first.
- To remove the app, delete `SotheOptimizer.exe`. If you enabled *Fixed Timer Resolution*, **revert that tweak first**
  (it removes the scheduled task and its copy in `Program Files`).
- Data lives in `%ProgramData%\SotheOptimizer` (backups, your own profiles, the log). The folder keeps its original
  name so installs from before the rename keep their backups.

## Layout

```
src/SotheOptimizer/
  SotheEngine.cs        apply / revert, LIFO write-ahead backups, registry + services + commands
  AppUpdater.cs         signed self-update          DataUpdater.cs   signed data (OTA, currently off)
  SignedData.cs         ECDSA verification          SecureStorage.cs hardened %ProgramData%
  TimerResolution.cs    ntdll timer + calibration   PowerShellRunner.cs  hidden PowerShell with real error detection
  data/*.json           tweaks, apps, features, profiles (embedded in the exe)
  MainWindow*.cs/xaml   WPF UI
tests/SotheOptimizer.Tests/   xUnit
tools/OtaSigner/              key generation and signing
tools/RoundTripCheck/         real apply/revert check with before/after system snapshots
```

## Contributing

Issues and pull requests are welcome; see [CONTRIBUTING.md](https://github.com/ImSothe/Toji-Windows-Optimizer/blob/main/CONTRIBUTING.md). Adding a tweak is editing
`tweaks.json`, and it must be undoable.

## Legal notice

Toji modifies deep Windows settings. **Use it at your own risk**: some tweaks disable security protections or can cause
instability. A restore point is created automatically; keep that option on. The software is provided "AS IS", with no
warranty. See [LICENSE](https://github.com/ImSothe/Toji-Windows-Optimizer/blob/main/LICENSE).

## License & credits

[MIT](https://github.com/ImSothe/Toji-Windows-Optimizer/blob/main/LICENSE). Built with AI-assisted development (Claude Code); the design, the decisions and the verification are
mine. Contact: [LinkedIn](https://www.linkedin.com/in/%C3%A1ngel-baz-monteiro-8196a21a9/).
