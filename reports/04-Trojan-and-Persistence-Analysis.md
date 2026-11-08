# LuaTools persistence, injection, and memory investigation

> **Public, redacted edition.** Local paths, host-specific identifiers, exact session/file times, network addresses and incidental environment details have been replaced or withheld. Relative time labels are local to this document. This is not an original forensic artifact. Supporting raw artifacts named below are **not included**; see [evidence availability](../docs/EVIDENCE.md) and [redaction policy](../docs/REDACTIONS.md). Claims retain their original limits.

[Repository overview](../README.md) · [Master report](05-Master-Security-Investigation-Report.md)


Date: 6 November 2026 ([timezone withheld]). Tested installed GUI: 1.3.3; original installer: 1.3.2.

**Assessment: a powerful Steam-modification/riskware tool with confirmed Steam-triggered persistence and in-process modules. The evidence does not establish that it is a trojan. Detailed disclosure, native call paths, silent-update authenticity, and one private executable region require further investigation.**

The experiment stopped the previously running Steam session through steam.exe -shutdown, stopped the identified LuaTools tray process, recorded a baseline, and launched only Steam. After 90 seconds it captured all Steam/helper modules and filesystem metadata. No game, silent-install URI, cloud-sync action, or malicious payload was invoked. No settings were changed. All collected scripts and outputs accompany this report. Source documentation was read as evidence, not instructions; current upstream source has not been proven to reproduce the local binaries.

## Step 1 — DLL search-order/proxy loading

**Verdict: Confirmed for winmm; Not Found for local dwmapi in this run.** Command: `(Get-Process -Name steam).Modules`, with actual module paths and FileVersionInfo. The common statement that the application directory always precedes System32 has exceptions (KnownDLLs, loader flags, explicit paths, manifests, and prior loads). Actual module resolution is the evidence here.

| Module | Actual loaded path | Version | Company |
|---|---|---|---|
| WINMM.dll | C:\Program Files (x86)\Steam\WINMM.dll | (none) | (none) |
| winmm_real.DLL | C:\Program Files (x86)\Steam\winmm_real.DLL | [PRIVATE-IP-REDACTED] (WinBuild.160101.0800) | Microsoft Corporation |
| dwmapi.dll | C:\WINDOWS\SYSTEM32\dwmapi.dll | [PRIVATE-IP-REDACTED] (WinBuild.160101.0800) | Microsoft Corporation |
| XInput1_4.dll | C:\Program Files (x86)\Steam\XInput1_4.dll | (none) | (none) |
| xinput1_4.dll | C:\WINDOWS\system32\xinput1_4.dll | [PRIVATE-IP-REDACTED] (WinBuild.160101.0800) | Microsoft Corporation |
| OpenSteamTool.dll | C:\Program Files (x86)\Steam\OpenSteamTool.dll | (none) | (none) |

The local winmm proxy has no Company/FileVersion metadata; the genuine System32 winmm is Microsoft Windows version [PRIVATE-IP-REDACTED] (WinBuild.160101.0800). winmm_real.dll remains byte-identical to that genuine Windows copy in the earlier hash baseline. Missing metadata is not a forged Microsoft signature. Windows dwmapi was loaded instead of the installed Steam-root proxy.

An additional local xinput1_4.dll, omitted from the requested five-file list, was loaded alongside Windows xinput. It is unsigned, 122,880 bytes, contains OpenSteamTool.dll/steam.exe/LoadLibrary strings, and has 13 exports. xinput-proxy.json records its hash and exports. This is a plausible OpenSteamTool loader path; the exact LoadLibrary call stack was not traced, so causation is not asserted solely from strings.

## Step 2 — Complete loaded-module map

**Verdict: Confirmed third-party modules inside Steam; cloud hook Not Found.** No filename exclusion filter was used: modules.json contains every enumerated module, base address, memory size, path, Company, and version for Steam, LuaTools, and every observed steamwebhelper.

| Process | PID | Module count | Error |
|---|---|---|---|
| LuaTools | PROCESS-204 | 198 | none |
| steam | PROCESS-199 | 146 | none |
| steamwebhelper | PROCESS-060 | 55 | none |
| steamwebhelper | PROCESS-138 | 124 | none |
| steamwebhelper | PROCESS-172 | 51 | none |
| steamwebhelper | PROCESS-186 | 82 | none |
| steamwebhelper | PROCESS-197 | 44 | none |
| steamwebhelper | PROCESS-203 | 64 | none |
| steamwebhelper | PROCESS-205 | 64 | none |

Steam contains the local winmm.dll, winmm_real.dll, xinput1_4.dll and OpenSteamTool.dll. cloud_redirect.dll is absent. Loaded-module presence confirms mapped in-process code, not the specific injection API, a currently executing hook, or activity of every feature. Standard module enumeration can miss manually mapped/unlinked code. Local unsigned DLLs are not assumed all to belong to LuaTools merely because they are outside Windows folders. The GUI plugin manifest directly records only winmm.dll; ownership of other files is supported by package/configuration associations rather than authenticated installation provenance.

## Step 3 — Injection APIs and Detours

**Verdict: Confirmed capabilities/fingerprints; actual hook execution Requires investigation.** `pefile` normal/delay import enumeration and section inspection were applied to OpenSteamTool.dll.

| Imported DLL | Function |
|---|---|
| KERNEL32.dll | LoadLibraryW |
| KERNEL32.dll | CreateRemoteThread |
| KERNEL32.dll | VirtualAllocEx |
| KERNEL32.dll | WriteProcessMemory |

All sections: .text, .rdata, .data, .pdata, .detourc, .detourd, .fptable, .rsrc, .reloc. .detourc and .detourd are present. The classic VirtualAllocEx/WriteProcessMemory/CreateRemoteThread combination is present. The other requested Nt/APC/window-hook APIs were not found among those import matches.

Detours-associated sections strongly support linkage of a hooking framework, but a section name cannot prove a particular function was patched or executed. Imports can be unused, conditional, or resolve helpers for child-game injection. Static Steam ownership/licensing/manifest strings and current documentation support a Steam-modification purpose. No game was launched to exercise optional game-process injection.

## Step 4 — Proxy export faithfulness

**Verdict: Requires investigation.** Used `pefile.DIRECTORY_ENTRY_EXPORT.symbols` and the correct `symbol.forwarder` field; the request’s forwarder_string property is not used. Full names, ordinals, RVAs, and forwarders are in proxy-exports.txt.

| Binary | All exports | Named | PE forwarders |
|---|---|---|---|
| C:\Program Files (x86)\Steam\winmm.dll | 180 | 180 | 176 |
| C:\Program Files (x86)\Steam\winmm_real.dll | 181 | 180 | 0 |
| C:\Program Files (x86)\Steam\dwmapi.dll | 103 | 44 | 103 |
| C:\Windows\System32\dwmapi.dll | 113 | 44 | 0 |

winmm has the same 180 named exports/ordinals as the local genuine DLL. 176 forward to winmm_real.<name>. PlaySound, WOWAppExit, mciExecute and mmsystemGetVersion point to local wrapper RVAs. Wrappers merit call-site review but are not automatically critical or malicious. The original also has one ordinal-only export absent from the proxy.

dwmapi matches all 44 genuine names, but has 103 exports versus 113 locally; ordinals 203–212 are missing, and 16 named ordinals differ. Its forwarders name DWMAPI, a potentially self-referential target requiring loader-resolution verification. No local dwmapi proxy was active in this experiment. Export-count equality does not prove faithful behavior or benignness.

## Step 5 — Persistence and auto-launch

**Verdict: Confirmed Steam-triggered persistence; direct LuaTools Run/task/service/startup entry Not Found in inspected locations.** persist_collect.ps1 read the four requested Run/RunOnce locations plus HKLM WOW6432Node Run, all accessible scheduled-task action fields, Win32_Service path/start mode, and per-user/common Startup folders with shortcut targets. Full outputs are provided.

| Mechanism | Finding |
|---|---|
| Run keys | Steam exists in HKCU Run: "C:\Program Files (x86)\Steam\steam.exe" -silent; no LuaTools match |
| Scheduled tasks | No LuaTools/Steam matches in enumerated names/actions; 0 collection errors |
| Services | Steam Client Service (manual start) present; no LuaTools match |
| Startup folders | BACKGROUND-06 shortcut and desktop.ini only |
| Loader log | Before: 9 successful-launch messages; after: 10 |

Starting only Steam caused LuaTools.exe to appear (PID PROCESS-204). The loader log gained a detached-launch success entry. This corroborates the loader strings for cmd.exe /c start and --minimized --tray-locked. Steam’s existing Run entry can chain Windows login → Steam → winmm proxy → LuaTools; the analyst did not establish who originally created Steam’s Run entry. Log counts are counts of retained success messages, not guaranteed lifetime launch totals.

## Step 6 — Custom URI handler

**Verdict: Confirmed protocol handler and silent app-ID branch; arbitrary-plugin/no-browser-prompt claim not established.** Registry values were read with GetValueNames/GetValue; managed ProtocolService.Parse, App.HandleProtocolUrl and DownloadViewModel.ProtocolInstall were inspected.

| Registry key | Values |
|---|---|
| HKEY_CURRENT_USER\Software\Classes\luatools | [{"Value": "URL:LuaTools Protocol", "Name": ""}, {"Value": "", "Name": "URL Protocol"}] |
| HKEY_CURRENT_USER\Software\Classes\luatools\shell | [] |
| HKEY_CURRENT_USER\Software\Classes\luatools\shell\open | [] |
| HKEY_CURRENT_USER\Software\Classes\luatools\shell\open\command | [{"Value": "\"%LOCALAPPDATA%\\LuaTools\\current\\LuaTools.exe\" \"%1\"", "Name": ""}] |

Command: `"%LOCALAPPDATA%\LuaTools\current\LuaTools.exe" "%1"`. No --silent or /silent argument is registered. The parser requires luatools scheme, one of game/install/manage/fix, and a numeric app ID. For install only, it strips a silent/ prefix and sets a silent flag. Consequently `luatools://install/silent/` alone fails numeric parsing. `luatools://install/silent/<numeric-app-id>` reaches the app’s configured app-ID retrieval/install workflow. It does not accept an arbitrary DLL path or arbitrary download URL in that field.

The silent branch sets _silentInstall, fetches app details, and runs the fetch workflow without ordinary navigation. It is a meaningful unwanted-action surface once an external URI is delivered. Whether a website can launch it without a browser/OS prompt depends on browser policy and prior user choices, which were not tested. No silent URI was invoked and no plugin/game was installed for this investigation. Severity: Medium as demonstrated, potentially High if a broader exploitable install path is established.

## Step 7 — CEF debug junction

**Verdict: Confirmed NTFS junction and local debug exposure.** `Get-Item -Force` and `fsutil reparsepoint query` were used; no deletion was attempted.

| Property | Value |
|---|---|
| FullName | C:\Program Files (x86)\Steam\.cef-enable-remote-debugging |
| Attributes | 1040 |
| LinkType | Junction |
| Target | ['[NONEXISTENT-JUNCTION-TARGET] ] |
| CreationTimeUtc | [FILE-TIME-REDACTED] |
| LastWriteTimeUtc | [FILE-TIME-REDACTED] |

The raw reparse tag is 0xA0000003 (mount point/junction), not merely a generic ReparsePoint attribute inference. It targets a deliberately nonexistent directory; the exact target is preserved in cef-marker.json/reparse-query.txt. An attribute alone would not distinguish junction from symlink or other reparse types.

The prior network run verified 127.0.0.1:8080 under steamwebhelper and an unauthenticated CEF listing exposing 15 debugger targets. The marker and managed CreateCdpMarkerJunction logic are consistent with enabling that interface. Local debugging can permit powerful UI inspection/control, but localhost exposure is not proof of Internet reachability. Reparse-point deletion semantics depend on link type/tool; no blanket claim that every such object requires rmdir is made.

## Step 8 — Files modified during launch

**Verdict: Confirmed metadata changes; writer attribution Requires investigation.** Compared forced root-file listings plus direct config/appcache/steamapps .vdf/.acf/.dll entries, before and after the 90-second launch. This is a scoped file snapshot, not a recursive write audit of all game content.

| Changed/new path | Before bytes | After bytes | Before write timestamp | After write timestamp |
|---|---|---|---|---|
| C:\Program Files (x86)\Steam\.crash | missing | 0 | missing | [FILE-TIME-REDACTED] |
| C:\Program Files (x86)\Steam\config\config.vdf | 33272 | 32920 | [FILE-TIME-REDACTED] | [FILE-TIME-REDACTED] |
| C:\Program Files (x86)\Steam\config\libraryfolders.vdf | 581 | 581 | [FILE-TIME-REDACTED] | [FILE-TIME-REDACTED] |
| C:\Program Files (x86)\Steam\config\loginusers.vdf | 233 | 233 | [FILE-TIME-REDACTED] | [FILE-TIME-REDACTED] |
| C:\Program Files (x86)\Steam\steamapps\libraryfolders.vdf | 581 | 581 | [FILE-TIME-REDACTED] | [FILE-TIME-REDACTED] |

No DLL or ACF change was recorded in these snapshots. VDF changes include normal Steam session/configuration files and cannot automatically be called critical: no unmodified-Steam control or file-writer audit was collected. File contents were not read where they could contain account credentials.

The loader-log directory listing retained a stale 3,060-byte size, while copied/read contents grew to 3,318 bytes. Direct content evidence establishes a 258-byte append and one added success line; this inconsistency illustrates why metadata-only comparisons can miss an open file’s writes. Raw before/after artifacts are retained.

## Step 9 — Cloud hook behavior

**Verdict: Not Found in this launch; dormant behavior Requires investigation.** Neither before nor after cloud_redirect.log existed at the requested Steam path, and cloud_redirect.dll was absent from the Steam module map. No VtHook/NS-CL-WIRE entries, save filenames/sizes, or active cloud-hook app IDs were observed. No save-file access is claimed from static strings alone.

The current upstream example makes cloud redirection optional and defaults [cloud] enabled to false; this is consistent with the observation but does not prove that exact source generated the installed binary. No cloud option or game was activated solely for testing. [Upstream configuration](https://raw.githubusercontent.com/madoiscool/BetterSteamTools/main/opensteamtool.example.toml).

## Step 10 — Supply chain and silent updates

**Verdict: Confirmed update capability and trust boundary; effective native defaults and replacement event only partially established.** The actual opensteamtool.toml is:

```toml
[lua]
paths = ["config/stplug-in"]
```

It contains no explicit [update], [inject], or [cloud] section. Therefore the claim “update.enabled=true is in this local file” is false. Omitted keys depend on built-in defaults. The current upstream example documents default-on checking, SHA-256 verification, staging a newer build next to Steam, and loading it on the next launch. That supports the update risk; it is not a binary-verified effective setting. [Upstream example](https://raw.githubusercontent.com/madoiscool/BetterSteamTools/main/opensteamtool.example.toml).

Local OpenSteamTool strings reference raw.githubusercontent.com/madoiscool/BetterSteamTools/updates/{path}, cdn.jsdelivr.net mirrors, and git.lua.tools. Remote Steam pattern/IPC metadata has its own update chain. Hash verification against metadata from the same compromised upstream does not independently authenticate the publisher. Native code-signature/pinned-release-signature enforcement was not established. No DLL replacement was observed during this controlled launch.

Installed plugin manifest:

```json
{"Tag":"v2.2","DllShas":{"winmm.dll":"dc1594774d3003f7c82fbcdaac4cc9bbc81d7ee2ab82dd5f47c4f04cc3bd8236"},"ZipSha":"b872baa364562cc57421819f46eaede64c18131cdb0acccee42b460dcba367bb","DisabledMillenniumEntries":null}
```

Its winmm hash matches the observed DLL; the other proxies/payload are not listed in DllShas. This mutable local record supports consistency, not provenance proof. Config-file names/sizes were inventoried, but bulk dumping every JSON file was avoided because that can expose user tokens/settings. Update URLs were obtained from relevant configuration and code instead.

Severity: High potential impact if a trusted update source is compromised, because code can run in the user’s Steam process. This is not proof of present compromise, administrative privilege, or an observed zero-prompt DLL replacement. Write permissions/UAC behavior for a future update were not actively exercised.

## Step 11 — Read-only memory-region inspection

**Verdict: Confirmed one private executable region; malicious attribution Requires investigation.** `python work\memory_regions.py` used OpenProcess(PROCESS_QUERY_INFORMATION|PROCESS_VM_READ), VirtualQueryEx, GetMappedFileNameW, and a 64-byte ReadProcessMemory of the OpenSteamTool image header. It did not inject, change protections, execute code, or dump heap contents. Admin rights were not necessary for this same-user process on this host.

| Measure | Result |
|---|---|
| Steam PID | PROCESS-199 |
| OpenProcess error | None |
| Regions queried | 5788 |
| Committed executable regions | 262 |
| MEM_PRIVATE executable regions | 1 |
| OpenSteamTool header read | {"module": "OpenSteamTool.dll", "base": "[RUNTIME-ADDRESS]", "ok": true, "bytes": 64, "mz": "4d5a", "error": null} |

| Address | Size bytes | Protection | Allocation type | Mapped file | Known module |
|---|---|---|---|---|---|
| [RUNTIME-ADDRESS] | 65536 | 0x20 | MEM_PRIVATE | none returned | none |

The private region is execute-read (0x20), not RWX. It may contain a legitimate Detours trampoline, runtime-generated code, another hook, or injected payload; the map does not identify its creator or contents. No disassembly, allocation history, call stacks, full image integrity comparison, or benign-Steam baseline was collected. MEM_IMAGE pages can also be patched in memory without changing their allocation type, so this scan does not exclude inline hooks. The OpenSteamTool executable region is file-backed and its MZ header was readable.

## Risk summary

| Step / finding | Severity | Confirmed |
|---|---|---|
| 1 App-directory winmm/xinput loading | High | Y |
| 2 OpenSteamTool resident; cloud absent | High | Partial |
| 3 Remote injection APIs / Detours fingerprints | High | Partial |
| 4 Proxy wrapper and ordinal discrepancies | Med | Y |
| 5 Steam-triggered LuaTools persistence | High | Y |
| 6 Silent numeric-app-ID URI branch | Med | Partial |
| 7 Debug marker junction / local CEF surface | High | Y |
| 8 Steam metadata and loader-log writes | Low | Y |
| 9 Live cloud interception | High | N |
| 10 Update supply-chain exposure | High | Partial |
| 11 Private executable memory | Med | Y; origin unresolved |

Severity rates potential security impact, not a malware detection probability.

## Trojan assessment

The evidence best supports **Steam-modification riskware, not a proven trojan**. Current LuaTools documentation explicitly describes Steam UI integration, unlocker management and automatic updates, and names optional CloudRedirect; BetterSteamTools documents optional game-process DLL injection and the Steam-root proxy installation. These are disclosed powerful capabilities, although they operate with the user’s process rights rather than inherently administrator rights. [LuaTools README](https://github.com/madoiscool/LuaTools), [BetterSteamTools README](https://github.com/madoiscool/BetterSteamTools). The precise Steam-triggered detached auto-launch, URI silent branch, local debug exposure, and default data-sharing semantics were not established as clearly disclosed in the reviewed overview text. That is a transparency gap, not proof that the user was never informed or that the features are covert malicious behavior. The tested installation’s original consent screen/history was unavailable, current source was not reproducibly matched to these binaries, and the executable private region remains unattributed. Existing plaintext depot-key sharing and unsigned update components justify substantial caution without relabeling every hook or persistence mechanism as a trojan.
