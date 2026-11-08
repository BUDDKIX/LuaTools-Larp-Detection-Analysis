# LuaTools 1.3.2 — Master Security Investigation Report

> **Public, redacted edition.** Local paths, host-specific identifiers, exact session/file times, network addresses and incidental environment details have been replaced or withheld. Relative time labels are local to this document. This is not an original forensic artifact. Supporting raw artifacts named below are **not included**; see [evidence availability](../docs/EVIDENCE.md) and [redaction policy](../docs/REDACTIONS.md). Claims retain their original limits.

[Repository overview](../README.md) · [Master report](05-Master-Security-Investigation-Report.md)


**Compiled:** 8 November 2026 (public edition; exact compilation time withheld)

**Target:** LuaTools 1.3.2 — Steam modification tool; the supplied installer was 1.3.2, but the installed GUI actually tested was **1.3.3**. Plugin record: v2.2; CloudRedirect GUI: 2.6.6.0.

I performed this analysis for one of [OneShed](https://www.youtube.com/@OneShed)’s video essays.

**Analysis attribution (analyst-provided):** I did the analysis using **ChatGPT Codex with the GPT-6 Astra model**, with **medium-to-high reasoning effort** and **full access to my PC**.

This is my account of the setup I used; the collection records do not independently verify the model or reasoning settings. Full access refers to Codex’s access permissions, not Windows administrator privileges: the recorded tests ran without elevation, and some tracing was unavailable.

**Collection tools:** Codex desktop agent using local PowerShell, Python/pefile, .NET metadata reader, Windows counters/API queries, and primary-source documentation.

**Test machine:** Windows; exact edition/version and OS build were **not recorded** in the source reports. AMD64 target processes/binaries and integrated graphics were recorded; user context was medium-integrity/non-elevated. **Antivirus installation/enabled status was not established.** The Windows DLL file version [PRIVATE-IP-REDACTED] is not asserted to be the machine’s OS build.

**Scope:** Static binary analysis and live behavioral analysis across four categories, 38 numbered steps plus baseline/subchecks (40+ checks). Some checks were inconclusive or unavailable. Original observations: **6 November 2026**; this compilation performs no retest.

Evidence convention: **T1** = Static Malware Analysis (9 steps); **T2** = GPU Miner Detection (8 steps); **T3** = Credential Theft and Network Analysis (10 steps); **T4** = Trojan and Persistence Analysis (11 steps). Citations identify the specific source step. The reports’ accompanying artifacts supply exact values and endpoint/timeline rows already within their evidence scope. “Clean” below means a negative result for a named test, not certification of the entire application. Proposed recommendations in Section 9 are not additional observed findings.

**Prompt workflow:** Four test prompts were used, one per investigation, followed by a fifth compilation prompt. Requested steps are not evidence of completed checks. For future analysis of LuaTools or other unfamiliar files, use VirtualBox or another properly isolated environment rather than an everyday PC. This was not the original test environment. See [prompts and safer testing](../docs/PROMPTS-AND-SAFE-TESTING.md).

## How to read this report

Start with the conclusion, then check its limitations. The **Source** column and references in square brackets tell you where the supporting evidence was reported; they are not extra findings.

| Reference | Report to open |
|---|---|
| **T1** | [Static malware analysis](01-Static-Malware-Analysis.md) |
| **T2** | [Mining investigation](02-GPU-Miner-Detection.md) |
| **T3** | [Credentials and network analysis](03-Credential-Theft-and-Network-Analysis.md) |
| **T4** | [Persistence and injection analysis](04-Trojan-and-Persistence-Analysis.md) |

- **§** means a section or numbered step; **§§** means several. In references to T1–T4, look for the numbered test steps in the linked report.
- **T1 §4** means “Test 1, Step 4.” **T3 §§1–2, 8** means “Test 3, Steps 1, 2 and 8.”
- A dash gives an inclusive range: **T2–T4** means Tests 2, 3 and 4. A semicolon separates references: **T3 §6; T4 §7** means Test 3, Step 6, and Test 4, Step 7.
- **[T1, Step 4]** is another way of writing **T1 §4**. Open the report link above, then find that step heading.

**Confirmed/observed** means the specific stated finding was supported. **Code confirmed** means a capability was found in code, not necessarily seen running. **Not observed/not found/clean for a named check** means that check found no evidence within its limits; it does not prove the whole program safe. **Unresolved/requires investigation** means there is not enough evidence to decide. **Suspicious** or a **High/Medium/Low** risk rating describes concern, not proof of malware.

Labels such as **REMOTE-IP-01** replace withheld details for privacy. **T+00:01:00** means one minute after that document’s reference point; it is not a wall-clock time and should not be matched across reports. Raw logs named in the text are not included publicly; see [evidence availability](../docs/EVIDENCE.md).

## 1. Executive Summary

The evidence supports **riskware**, with no conclusive malware or trojan verdict and no cryptocurrency mining observed during the tested run. The most significant findings are **default-enabled plaintext Steam depot-key sharing in LuaTools.dll**, **unsigned Steam-resident proxies/OpenSteamTool with reproducible Steam-triggered LuaTools auto-launch**, and **multiple component update sources with incomplete independent provenance assurance**. The packing/RWX checks were clean for their tested indicators, and the bounded mining test found neither sustained mining-like resource use nor requested mining-port traffic. Browser credential theft remains unresolved because file reads and encrypted network payloads were not traced. Local CEF debugging exposure and an unattributed 64 KB private executable region add security concerns, but neither establishes a hidden malicious payload. The live cloud-hook path was inactive and the installed GUI was 1.3.3, so these results do not establish all behavior of every component or of the original 1.3.2 installer alone. [T1, Steps 3–4, 7; T2, Steps 1–8; T3, Steps 6–10; T4, Steps 5, 10–11]

## 2. Component Overview

| File | Origin / Repo association | Signature | Role | Source Available? |
|---|---|---|---|---|
| LuaTools.exe | LuaTools desktop package; madoiscool/LuaTools | Unsigned | Native .NET apphost for GUI | Public GUI repository observed; exact binary/source match not established |
| LuaTools.dll | madoiscool/LuaTools | Unsigned | Managed WPF logic, auth, analytics, install/update workflows | Public repository observed; installed IL also inspected |
| winmm.dll | LuaTools loader / LTSP release reference | Unsigned | Proxy and detached GUI launcher | Public source availability not established; do not claim unavailable |
| winmm_real.dll | Microsoft Windows; local System32 copy comparison | Valid Microsoft | Renamed genuine multimedia library | Not established in the reports |
| dwmapi.dll | BetterSteamTools/OpenSteamTool-associated proxy | Unsigned | Alternative loader/proxy; dormant in observed Steam run | Related public repository observed; exact binary provenance unverified |
| OpenSteamTool.dll | madoiscool/BetterSteamTools | Unsigned | Steam modification, hooks, optional remote-process injection | Public repository/documentation observed; not reproducibly matched |
| cloud_redirect.dll | CloudRedirect; madoiscool/Selectively11 references in component chain | Unsigned | Optional cloud save/RPC redirection and token storage | Public source availability for exact installed native build not established |
| CloudRedirect.exe | madoiscool download reference; Selectively11 updater reference | Unsigned | Bundled .NET companion/provider GUI | Repository references known; exact build/source equivalence unverified |

Source: T1 Steps 1, 3–4, 7; T4 Steps 1–3, 9–10 and Trojan assessment. **Neither winmm.dll nor cloud_redirect.dll was conclusively shown to lack publicly available source at test time.** The evidence establishes a source-verification gap, not public nonexistence. An additional unsigned Steam-root **xinput1_4.dll** was discovered in T4 Step 1 and must be included in exposure/removal review even though it is outside this eight-file table.

## 3. Findings by Test

### 3.1 Static Malware Analysis

All eight installed targets and the installer existed. Seven installed targets and the installer were unsigned; winmm_real.dll had a valid Microsoft signature and matched the genuine System32 file byte-for-byte. Exact SHA-256 identifiers follow; the wider inventory included all DLLs recursively under the GUI directory and extracted installer/bundle members. [T1, Steps 1, 3] 

| File | SHA-256 |
|---|---|
| LuaTools-win-Setup.exe | BAE35A2C7FB871ADF5AEC2A4F8BADE951533A1FB8672B273986D57CDDF58EE8F |
| LuaTools.exe | F88AAFD626484943D524A6A4E355633DD8E52B8AF698816B0A469B73C7495A45 |
| LuaTools.dll | B5792220F2944B9E3BE94ABFCF55D6D50F1040E7AA1B17965F92D777E6725972 |
| winmm.dll | DC1594774D3003F7C82FBCDAAC4CC9BBC81D7EE2AB82DD5F47C4F04CC3BD8236 |
| winmm_real.dll | 308DD7E06A8925E011D1D3DECDE646502EF12E906ECA830DAB5BE43A7F64660D |
| dwmapi.dll | 8E42F465C276C4D0430FB24158724979605B794A414BA4D1803AD66DD289411B |
| OpenSteamTool.dll | 24A6841E2E484D4586CEC5344F71B9913B81692BB0E8FF557B472D565F5DFDFE |
| cloud_redirect.dll | E0FA412D1DC41C40AFF0F733C5C42DD2C004A8FAFCE355AE2EF13B5541349996 |
| CloudRedirect.exe | 5A15983D6EB435E80D1375F3D013E87F4DD86D3618FB269EF0C64E2A1F1490A8 |

No scanned original/extracted binary had RWX sections, and no non-resource section exceeded entropy 7.2. CloudRedirect.exe’s .rsrc entropy was 7.957, but its 9,354,647-byte overlay was a valid .NET bundle; the installer’s 7,325,114-byte overlay was ZIP data. winmm_real.dll’s 11,696-byte overlay was its certificate table. Embedded CloudRedirect native/managed payloads were extracted and inspected; its embedded cloud_redirect.dll differed from the installed Steam copy, which demonstrates component divergence rather than proven tampering. [T1, Step 3]

OpenSteamTool.dll imported VirtualAllocEx, WriteProcessMemory and CreateRemoteThread and contained Detours-associated sections. cloud_redirect.dll imported both CryptProtectData and CryptUnprotectData with cloud/provider-token context. Managed LuaTools code confirmed that a null DonateKeys setting defaults to true, startup invokes the donation method, and Steam config\config.vdf DecryptionKey values can be POSTed as ID:key pairs to http://REMOTE-IP-09/donatekeys/send. This is content-decryption-key sharing, not demonstrated account-password theft; live transmission was not established. [T1, Step 4]

Resolved lexical false positives included ethash inside GetHashCode/AssetHash/CryptGetHashParam; pool in thread/connection/array pools and UI resources; wallet/Cookies icon names or Steam HTTP APIs; miner inside French terminer; nonce protocol vocabulary; donate referring to depot keys; version-shaped IP candidates; XML/XAML schema identifiers; and “preserve local state” cloud-sync logging rather than Chromium’s Local State file. The requested explicit mining/C2/theft-tool indicators did not support a miner or browser stealer. [T1, Step 6; T3, Step 1]

The installer had ZoneId=3 and HostUrl=https://lua.tools/ with a trailing NUL; CloudRedirect.exe had no readable web-mark stream. Neither result establishes historical SmartScreen execution. Installer manifest/properties said 1.3.2, installed GUI properties 1.3.3.0 and sq.version 1.3.3; this supports version divergence with update capability, not proof of which installation mechanism occurred. Plugin installed.json recorded v2.2 and winmm hash DC1594774D3003F7C82FBCDAAC4CC9BBC81D7EE2AB82DD5F47C4F04CC3BD8236, which matched disk; other Steam DLLs were not covered by DllShas. The local record is mutable and not an authenticated publisher baseline. [T1, Steps 2, 7–8]

**Verdict: Suspicious.** **Most significant finding:** default-enabled plaintext depot-key sharing in LuaTools.dll is established at code level, while actual upload was not observed. [T1, Step 4]

### 3.2 GPU Miner Detection

The seven mining-scan targets contained no requested GPU-compute library/API strings or imports, and all mining lexical hits were contextualized rather than labeled malware. LuaTools alone loaded 235 modules without a compute-name match; 64 total target module snapshots across the test produced zero compute matches and zero access errors. Conventional module enumeration still cannot exclude manually mapped code, transient loads, graphics-shader computation, or later downloads. [T2, Steps 1–4]

The continuous GPU collector covered **601.975 seconds**, with one warm-up plus **40 interval samples**. LuaTools busiest-engine GPU utilization averaged **1.9700%**, peaking at **2.5920%** over 15-second intervals; supplementary one-second snapshots peaked at **5.1497%**. The GPU-active steamwebhelper averaged **0.0534%**, interval peak **1.3458%**. No GPU instance was reported for Steam and some helpers, so their value was unavailable rather than a measured zero. Target activity was on the 3D engine; background capture/desktop applications were already active. [T2, Step 5]

Thirty scheduled network polls produced zero requested mining-port hits. CPU sampling used elapsed CPU-time deltas normalized to the recorded logical processor count, not cumulative seconds mislabeled as CPU%. LuaTools averaged **1.3589% CPU**, interval peak **1.8368%**; Steam averaged **0.0848%**, peak **0.1832%**, over 20 intervals. All seven target hashes were unchanged. GPU watts were unavailable; Task Manager was present, and encrypted/arbitrary-port traffic, brief bursts, dormant or observer-aware behavior were not excluded. [T2, Steps 5–8; Timing and evidence integrity]

**Verdict: Clean for mining evidence observed in this bounded run.** **Most significant finding:** the combined static, module, resource and port checks found no affirmative GPU/CPU mining evidence; they do not certify that mining is impossible. [T2, Steps 1–8]

### 3.3 Credential Theft and Network Behavior

The eight-file credential scan found no confirmed browser-theft, LSASS/tool, or clipboard-monitoring indicator. Hits resolved to UI Cookies icons, cloud-sync “local state” wording, and dual-use CryptUnprotectData. Native import-neighborhood strings are import-table data, not data-flow proof; both DPAPI encrypt/decrypt were imported. Managed CloudRedirect TokenFile methods and LuaTools AuthService methods protect/read their own token stores, including auth.dat, but no native decrypt-call tracer proved every input benign. [T3, Steps 1–2]

No requested webhook/tunnel/paste channel was found. Static code references numerous authentication, release, manifest, cloud-provider and mirror hosts, so the supplied small allowlist was not a complete legitimate-host list. The four-minute LuaTools-only run captured GitHub, analytics, and an extra db.lua.tools connection at REMOTE-IP-23:443; Steam-active monitoring added Steam/CDN/update/manifest and loopback endpoints. **Section 5 gives the full unique endpoint list from T3 and the additional nonstandard endpoints explicitly reported by T2, with retained ownership evidence.** Cache mappings are candidates rather than proven TLS SNI, and process ownership does not identify the initiating DLL. [T3, Steps 3–5; T2, Step 6]

Bridge GET / returned 404 with {"success":false,"error":"Not found"}; /loaded-apps returned 200 with {"success":true,"apps":[]}. Port 6767 was loopback under PID 4/HTTP.sys and returned wildcard CORS headers. CEF /json on loopback 8080 returned 200 and 15 page-debug targets with WebSocket debugger URLs; no debugger command was executed. cloud_redirect.log was absent in searched locations and the native cloud DLL was not observed loaded, so there was no live save-file enumeration evidence. [T3, Steps 6–7]

Credential-file write timestamps remained unchanged, which does not rule out reads; no reliable per-process read audit was available. LuaTools DoH code targeted 1.1.1.1/dns-query, but no sampled LuaTools connection to the supplied resolver IPs was observed. Steamwebhelper contacted 8.8.4.4:443, a DoH candidate rather than confirmed payload; LuaTools domains were present in the Windows cache. Over **301.769 seconds**, Wi-Fi sent **37,010,276 bytes** and received **30,908,557 bytes** for the whole machine; those values cannot be attributed to LuaTools/Steam or labeled exfiltration. TLS, short connections, UDP remote traffic, native reads and per-process volume remained unresolved. [T3, Steps 8–10]

**Verdict: Suspicious overall security exposure; credential theft Unresolved, not confirmed or refuted.** This classification retains the source report’s risk assessment and does not assert that the observed endpoint traffic was malicious. **Most significant finding:** browser credential access and encrypted data transfer could not be resolved despite explainable observed connections and unchanged metadata. [T3, Steps 4–10 and Conclusions]

### 3.4 Trojan and Persistence Analysis

Steam loaded local winmm.dll, winmm_real.dll, xinput1_4.dll and OpenSteamTool.dll, while using the genuine Windows dwmapi.dll; cloud_redirect.dll was absent. The full Steam map had 146 modules, with helper processes separately enumerated. Presence establishes mapped code, not which injection API ran or every hook’s activation. OpenSteamTool’s remote-write/thread APIs and .detourc/.detourd sections support hooking/injection capability, not malicious execution by themselves. [T4, Steps 1–3]

The winmm proxy had 180 named exports matching the local genuine names/ordinals: 176 forwarded to winmm_real and four used local wrappers (PlaySound, WOWAppExit, mciExecute, mmsystemGetVersion). The genuine DLL had an additional ordinal-only export. The dormant dwmapi proxy had all 44 names but 103 total exports versus 113 genuine, ten missing ordinals, 16 differing named ordinals and DWMAPI forwarders requiring resolution verification. Counts do not prove faithful implementation. [T4, Step 4]

A controlled launch of only Steam caused LuaTools to reappear. Retained successful detached-launch log entries increased **9 → 10**. No direct LuaTools Run/task/service/startup entry was found in inspected locations, but Steam’s own HKCU Run entry can chain login → Steam → proxy → LuaTools. The URI handler executes the quoted LuaTools.exe path with "%1", with no --silent argument; code recognizes install/silent/<numeric-app-ID>. A bare install/silent/ is rejected; arbitrary plugin URLs and browser-prompt bypass were not established. [T4, Steps 5–6]

The .cef-enable-remote-debugging marker was a confirmed NTFS mount-point/junction (tag 0xA0000003), consistent with local debug exposure. Launch snapshots recorded .crash and Steam config/library/login VDF metadata changes, not attributed malicious writes; no DLL/ACF changes were recorded. Directory metadata for the loader log was stale, but direct copied contents grew **3,060 → 3,318 bytes**, adding one success entry. No cloud log or actual save-file names/sizes were captured. [T4, Steps 7–9]

Local opensteamtool.toml contained only [lua] paths=["config/stplug-in"], not explicit update.enabled=true. Upstream configuration described default-on update checking, hash verification and next-launch staging, while installed code referenced GitHub/jsDelivr/git.lua.tools sources; exact native effective defaults, independent publisher authentication and a replacement event were not established. The memory scan succeeded without elevation, found **262 committed executable regions**, and identified **one 65,536-byte MEM_PRIVATE execute-read region**, with no file mapping or known module. It could be a trampoline/runtime/hook or payload; attribution and disassembly were not done. [T4, Steps 10–11]

**Verdict: Suspicious riskware behavior; a trojan is not established.** **Most significant finding:** Steam-triggered detached LuaTools launch was reproduced independently of a direct LuaTools startup entry. [T4, Step 5 and Trojan assessment]

## 4. Behavioral Timeline

All times below are relative time, 6 November 2026. This consolidates recorded event milestones; raw polls are retained in the source evidence. Exact timestamps were not recorded for every static check or memory query, so none are invented.

| Timestamp | Event | Process | Source (Test + Step) |
|---|---|---|---|
| 2026-11-06 (time unrecorded) | Static binary, signature, PE, IL and package analysis | Files only | T1 Steps 1–9 |
| T+00:00:00 | Baseline started; no target launch yet. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:00:28 | Baseline complete. Launching LuaTools. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:00:28 | LuaTools launch requested. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:01:17 | LuaTools-only snapshot complete after 45-second wait. Launching Steam. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:01:17 | Steam launch requested. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:02:22 | Ten-minute combined observation started. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:12:27 | Observation complete. GPU samples=40; network polls=30; CPU samples=20; elapsed seconds=604.7842564. | LuaTools / Steam / collector | T2 baseline, Steps 3–7 |
| T+00:25:23 | Resetting prior LuaTools tray process after graceful close did not exit it; Steam shutdown already requested. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:25:25 | Clean pre-launch baseline started. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:25:31 | LuaTools launch requested; initialization snapshots follow. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:26:04 | Four-minute LuaTools-only observation started. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:30:05 | LuaTools-only observation complete; launching Steam. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:31:07 | Combined observation started: 12 required polls over four minutes; extended to five minutes for adapter volume. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:36:09 | Scheduled observation complete; bridge probes and log review are performed separately. | LuaTools / Steam / collector | T3 baseline, Steps 4–5, 8, 10 |
| T+00:36:25 | http://127.0.0.1:6767/ returned 404 | Local HTTP/CEF | T3 Step 6 |
| T+00:36:25 | http://127.0.0.1:6767/loaded-apps returned 200 | Local HTTP/CEF | T3 Step 6 |
| T+00:36:25 | http://127.0.0.1:8080/json returned 200 | Local HTTP/CEF | T3 Step 6 |
| T+00:49:38 | Requesting Steam shutdown for fresh launch test. | Steam / LuaTools | T4 Steps 5, 8–9 |
| T+00:49:41 | Launching only Steam; LuaTools absent at prelaunch baseline. | Steam / LuaTools | T4 Steps 5, 8–9 |
| T+00:50:26 | 45 seconds elapsed. | Steam / LuaTools | T4 Steps 5, 8–9 |
| T+00:51:12 | 90-second launch observation complete. | Steam / LuaTools | T4 Steps 5, 8–9 |

After the 90-second Steam run, a read-only memory query found one private executable region; exact scan time not recorded. [T4, Step 11]

## 5. Network Endpoint Map

Scope: every unique **target-process** remote IP/port pair retained in T3’s endpoint inventory, plus the additional Steam and background nonstandard pairs explicitly reported in T2. Static-only endpoints are separated below. This is not a claim that polling captured every packet or that all unrelated machine endpoints were analyzed. Owner/network names are the original T3 registration results; failed lookups remain unresolved. All observations are TCP; HTTPS/HTTP is expected from ports/code, not payload-verified. Endpoint risk is left **unresolved** where the source did not assign an endpoint-specific severity. DNS-cache associations can be shared-IP candidates. [T3, Steps 4–5, 9 and Registry ownership; T2, Step 6]

| Endpoint / Domain; owner/network | IP | Port | Protocol | Purpose | Observed Live? | Risk Level | Source |
|---|---|---|---|---|---|---|---|
| [STEAM-SERVICE-HOST]; VC-AP | REMOTE-IP-01 | 27018 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST]; VC-AP | REMOTE-IP-01 | 27020 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| analytics.lua.tools, lua.tools; CLOUDFLARENET | REMOTE-IP-02 | 443 | TCP; HTTPS expected | Analytics / shared LuaTools hosting (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | 6767 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (LuaTools.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | 8080 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (LuaTools.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-01 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-02 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-03 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-04 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-05 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-06 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steam.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-07 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steam.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| loopback; local host | 127.0.0.1 | EPHEMERAL-PORT-08 | TCP; application protocol unverified | Local bridge/CEF/IPC; ephemeral endpoint direction not proven (steamwebhelper.exe) | Yes | Unresolved; local exposure concern | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST]; registration unresolved | REMOTE-IP-03 | 443 | TCP; HTTPS expected | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST]; registration unresolved | REMOTE-IP-03 | 27037 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST]; VALVE-L1 | REMOTE-IP-04 | 27032 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net; VALVE-L1 | REMOTE-IP-05 | 27028 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net; VALVE-L1 | REMOTE-IP-06 | 443 | TCP; HTTPS expected | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net; VALVE-L1 | REMOTE-IP-06 | 27021 | TCP; application protocol unverified | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| analytics.lua.tools, lua.tools; CLOUDFLARENET | REMOTE-IP-10 | 443 | TCP; HTTPS expected | Analytics / shared LuaTools hosting (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| manifest.luastools.xyz; CLOUDFLARENET | REMOTE-IP-11 | 443 | TCP; HTTPS expected | Plugin manifest service (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| raw.githubusercontent.com, release-assets.githubusercontent.com; US-GITHUB-20170413 | REMOTE-IP-12 | 443 | TCP; HTTPS expected | Release/source/update resources (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| raw.githubusercontent.com, release-assets.githubusercontent.com; US-GITHUB-20170413 | REMOTE-IP-13 | 443 | TCP; HTTPS expected | Release/source/update resources (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| hostname unresolved; SKYCA-3 | REMOTE-IP-14 | 443 | TCP; HTTPS expected | CDN/provider network; exact request purpose unresolved (steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| hostname unresolved; SKYCA-3 | REMOTE-IP-15 | 443 | TCP; HTTPS expected | CDN/provider network; exact request purpose unresolved (steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| client-update.fastly.steamstatic.com; SKYCA-3 | REMOTE-IP-16 | 443 | TCP; HTTPS expected | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| github.com; MSFT | REMOTE-IP-17 | 443 | TCP; HTTPS expected | Release/source/update resources (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| api.github.com; MSFT | REMOTE-IP-18 | 443 | TCP; HTTPS expected | Release/source/update resources (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| clientconfig.akamai.steamstatic.com, steamconnecttest.com; [REGISTRY-DETAIL-REDACTED] | REMOTE-IP-19 | 80 | TCP; HTTP expected | Steam service/CDN/connectivity (steam.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| api.steampowered.com; AKAMAI | REMOTE-IP-20 | 443 | TCP; HTTPS expected | Steam service/CDN/connectivity (steam.exe, steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| steamcommunity.com; AKAMAI | REMOTE-IP-21 | 443 | TCP; HTTPS expected | Steam service/CDN/connectivity (steam.exe, steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| hostname unresolved; AKAMAI | REMOTE-IP-22 | 443 | TCP; HTTPS expected | CDN/provider network; exact request purpose unresolved (steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| db.lua.tools; [REGISTRY-DETAIL-REDACTED] | REMOTE-IP-23 | 443 | TCP; HTTPS expected | Authentication/download-history code context (LuaTools.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5 |
| hostname unresolved; GOGL | 8.8.4.4 | 443 | TCP; HTTPS expected | Steam helper resolver-IP connection; DoH unconfirmed (steamwebhelper.exe) | Yes | Unresolved; no malicious payload shown | T3 Steps 4–5, 9 |
| Owner not verified; pre-existing BACKGROUND-01 traffic | REMOTE-IP-07 | 2096 | TCP; application protocol unverified | Background comparator; not attributed to LuaTools | Yes | Unresolved; not identified as mining | T2 Step 6; T3 Step 5 ownership evidence |
| Owner not verified; pre-existing BACKGROUND-01 traffic | REMOTE-IP-08 | 2053 | TCP; application protocol unverified | Background comparator; not attributed to LuaTools | Yes | Unresolved; not identified as mining | T2 Step 6; T3 Step 5 ownership evidence |
| VALVE-L1 (registration in T3) | REMOTE-IP-04 | 27019 | TCP; application protocol unverified | steam.exe; Steam connection | Yes | Unresolved; not identified as mining | T2 Step 6; T3 Step 5 ownership evidence |

Static-only security-relevant destination: **http://REMOTE-IP-09/donatekeys/send**; HTTP/80 code path, depot-key sharing, **not observed in sampled target connections**. This is a high-impact data-sharing concern, not an unknown-host/C2 inference. **https://1.1.1.1/dns-query** is a static DoH path; no LuaTools connection was observed. Other embedded mirror/provider URLs are retained in T1 Step 6 and T3 Step 3; their presence is not live contact. [T1 Step 4; T3 Steps 3, 9]

## 6. Risk Register

Statuses distinguish capability, observation and uncertainty. Required template assertions that conflict with the evidence are retained as unresolved items, not asserted facts.

| # | Risk Description | Severity | Status | Evidence Source |
|---|---|---|---|---|
| 1 | Default-enabled plaintext depot-key sharing in LuaTools.dll | High | Code confirmed; actual upload unobserved | T1 Step 4; T3 Step 3 |
| 2 | Unsigned Steam-resident winmm/xinput/OpenSteamTool components | High | Loaded confirmed; all injection call paths untraced | T4 Steps 1–3 |
| 3 | Multiple update sources / potential silent replacement | High | Capability documented; local effective native default and replacement event unproven | T1 Step 7; T4 Step 10 |
| 4 | Steam-triggered GUI persistence without direct LuaTools Run key | High | Reproduced; Steam itself has a Run key | T4 Step 5 |
| 5 | Silent numeric-app-ID URI workflow | Med | Code confirmed; arbitrary plugin/zero-browser-prompt claim unproven | T4 Step 6 |
| 6 | cloud_redirect.dll DPAPI decryption capability | Context-dependent | Own-token context supported; native inputs untraced | T3 Step 2 |
| 7 | CEF debug junction/local debug endpoints | High | Junction and live local debug listing confirmed | T4 Step 7; T3 Step 6 |
| 8 | winmm.dll/cloud_redirect.dll source not verified | Unrated limitation | Public unavailability NOT established; exact binary/source matching absent | T4 Steps 2, 9–10 and assessment |
| 9 | No antivirus on machine | Unrated limitation | NOT established; no signature-engine malware verdict supplied | T1 Step 1; T2 Step 8 scope; report-wide limitation |
| 10 | Four winmm wrappers / dwmapi ordinal and forwarding discrepancies | Med | Static confirmed; wrapper/loader correctness unresolved | T1 Step 5; T4 Step 4 |
| 11 | 64 KB private executable Steam region | Med | Observed; creator/content/intent unknown | T4 Step 11 |
| 12 | Wildcard-CORS loopback bridge | Suspicious exposure | Unauthenticated GET confirmed; cross-origin exploitation untested | T3 Step 6 |
| 13 | Unobserved browser credential-file reads | Unrated limitation | Read attribution unresolved despite unchanged write timestamps | T3 Step 8 |
| 14 | TLS/short-lived/UDP traffic and per-process volume gaps | Unrated limitation | Capture/attribution unavailable; host totals only | T3 Steps 4–5, 9–10 |
| 15 | Cloud interception/save access | High if activated | Not observed; native cloud path inactive | T4 Step 9; T3 Step 7 |
| 16 | Version and embedded/native component divergence | Requires investigation | Measured; tampering not established | T1 Steps 3, 7–8 |
| 17 | Observed VDF changes cannot be attributed to LuaTools | Low | Metadata changes confirmed; normal Steam writes possible | T4 Step 8 |
| 18 | Observer-aware/dormant mining cannot be excluded | Unrated limitation | Negative bounded test; Task Manager present, no GPU watts | T2 Steps 5–8 |
| 19 | Original installation disclosure/user consent unknown | Requires investigation | Public feature documentation exists; original user experience not recorded | T4 Steps 6, 10 and Trojan assessment |

## 7. Testing Limitations

| Unverified item / limitation | Reason and implication |
|---|---|
| Exact winmm/cloud_redirect source equivalence | Public-source absence was not proven. No reproducible source-to-binary match or complete native disassembly; native behavior remains partly unknown. |
| Antivirus and OS/account baseline | Antivirus status, exact OS edition/build and guest-only Steam status were not established. Do not assert no antivirus, a particular Windows build, or only a guest account. |
| Credential-file reads | No functioning per-process file/API trace. LastWriteTime changes or stability cannot prove reads or identify the reader. |
| Packet/TLS inspection | PktMon driver denied access in the non-elevated session; no TLS interception. Payloads, short-lived connections and UDP remotes are not fully characterized. |
| Privileges | Medium integrity was recorded; not every diagnostic requires admin. Read-only Steam memory queries succeeded, so do not report all memory forensics as unavailable. |
| Memory provenance | One private execute-read region was mapped but not disassembled/attributed; no allocation history or benign control. File-backed code could also be patched. |
| Mining horizon and sensors | Approximately ten-minute idle test on integrated graphics; Task Manager present, power telemetry unavailable, limited triggers. Negative findings do not exclude delayed/hardware-dependent/evasive mining. |
| Cloud/account/game conditions | No game/cloud path activated; no native cloud DLL/log observed. Account mode was not documented, and untested authenticated/game-specific paths remain gaps. |
| URI installation and update replacement | Not actively invoked/replaced during controlled tests. No confirmed arbitrary payload acceptance, browser prompt bypass, or fresh replacement-DLL behavior. |
| Update/disclosure provenance | Current documentation can differ from the installed build; original consent history was not available. Remote hashes alone do not independently authenticate a compromised publisher. |
| Network byte attribution | Adapter counters cover all software and virtual adapters can overlap; no justified per-process outbound-byte figure. |
| Filesystem snapshot scope | Launch comparison covered root and selected direct config/appcache/steamapps files, not every recursive game file; writer attribution unavailable and log metadata was stale. |
| Historical scope | Tests were on 6 November; no retest for this 8 November compilation. A future update can change behavior. |

Sources: T1 Steps 1–9; T2 Steps 4–8; T3 Steps 4–10; T4 Steps 2, 6, 8–11 and assessment.

## 8. Overall Verdict

**Malware:** a hidden malicious payload was not established; neither absence nor safety was proven. **Miner:** no affirmative GPU/CPU mining evidence was observed in the bounded run. **Trojan:** covert unauthorized malicious behavior was not established, and the original user’s awareness/consent could not be reconstructed. **Accurate classification:** Steam-modification **riskware** that loads unsigned local components into Steam, starts LuaTools through a Steam proxy, integrates with the Steam UI, uses remote update/authentication/manifest services, exposes local bridge/debug interfaces, and contains default-enabled plaintext sharing of depot content-decryption keys. Some powerful functionality is publicly described; that does not resolve the disclosure gaps, native memory/code uncertainty, inactive cloud path or encrypted network content. [T1 Step 4; T2 Steps 1–8; T3 Conclusions; T4 Steps 1–11 and assessment]

## 9. Recommendations

These are proposed risk-reduction and removal actions derived from the findings, **not actions executed in this compilation**. Removal completeness cannot be guaranteed from a partial installation inventory; preserve necessary data and verify actual paths first.

### If keeping the software

1. **Disable depot-key donation** explicitly in the app and verify the setting persists; do not treat a null value as disabled. This addresses the confirmed plaintext-sharing code path. [T1 Step 4]
2. **Review and restrict unattended updates and optional components** using documented controls. Record trusted versions/hashes and inspect replacements before allowing them to run; the installed TOML alone did not establish effective update defaults. [T4 Step 10]
3. **Reduce local debug and URI exposure** when not needed: disable the app’s Steam integration through supported controls and verify that 8080/6767 and automatic marker recreation stop. Review external-protocol launch permissions. [T3 Step 6; T4 Steps 6–7]
4. **Use a separate test environment for further evaluation** and avoid introducing additional sensitive data until native file access, cloud behavior and the private executable region are resolved. [T3 Steps 7–8; T4 Steps 9, 11]
5. **Complete the missing assurance checks:** establish antivirus protection/status, collect an authorized file/network trace where permissions allow, and repeat relevant checks after updates. An antivirus result alone would not resolve all behavioral risks. [T1 scope; T3 Steps 8–10; T4 Step 11]

### If removing the software

1. **Preserve the reports and back up settings/save data you need.** Close Steam, its helpers, LuaTools and CloudRedirect. Use the application’s supported uninstall/disable workflow if available, then inspect residuals; the tests did not validate that uninstaller’s completeness.
2. **Remove the identified third-party Steam-root components after verifying their paths and identity:**
   - `C:\Program Files (x86)\Steam\winmm.dll`
   - `C:\Program Files (x86)\Steam\winmm_real.dll` — the renamed copy placed beside Steam, not the genuine System32 file.
   - `C:\Program Files (x86)\Steam\dwmapi.dll`
   - `C:\Program Files (x86)\Steam\OpenSteamTool.dll`
   - `C:\Program Files (x86)\Steam\cloud_redirect.dll`
   - `C:\Program Files (x86)\Steam\xinput1_4.dll` — the confirmed unsigned local proxy.
   Review `opensteamtool.toml`, loader logs, cloud logs if present and `config\stplug-in` content as application-specific residuals. Preserve unrelated scripts, games, saves and Steam configuration. Do not delete Windows System32/SysWOW64 counterparts or treat every Steam-folder DLL as LuaTools-owned. [T1 Step 1; T4 Steps 1–2, 8–10]
3. **Remove only the verified junction itself**, not its target. Confirm `.cef-enable-remote-debugging` still has the recorded junction type; from Command Prompt, the non-recursive command is `rmdir "C:\Program Files (x86)\Steam\.cef-enable-remote-debugging"` (no `/s`, no `del`). Do not enumerate or delete the target directory. Re-check that integration software does not recreate it. [T4 Step 7]
4. **Remove the confirmed per-user URI registration:** `HKEY_CURRENT_USER\Software\Classes\luatools`, including `shell\open\command`, after exporting it if rollback is desired. No direct LuaTools Run/RunOnce/task/service entry was found, so no additional such key is asserted here. Keep Steam’s `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` Steam value unless you separately want to disable Steam login startup. Do not remove the genuine Steam Client Service. [T4 Steps 5–6]
5. **Remove the application data/program folders once required user data is preserved:** `%LOCALAPPDATA%\LuaTools`, `%APPDATA%\LuaToolsGui` (including plugin/cloudredirect subfolders), and `%APPDATA%\CloudRedirect`. These may contain settings and provider authentication data; deletion is irreversible without a backup. Also remove the original installer if no longer needed. This list covers paths documented by the investigations, not a proven inventory of every possible cache/residual. [T1 Steps 1, 7–8; T3 Steps 2, 7; T4 Step 10]
6. **Verify after removal:** launch Steam; confirm LuaTools does not auto-start, the removed local proxies/OpenSteamTool/cloud_redirect are absent from Steam’s module list, normal Windows winmm/dwmapi/xinput resolution is used, the URI key and junction stay absent, and the previously exposed local endpoints are no longer present because of this integration. Inspect remaining scheduled/startup entries and use the supported Steam repair/reinstall route if necessary. Never replace genuine Windows DLLs manually with downloaded copies. These checks establish removal of observed integration, not forensic proof that the whole machine is clean. [T4 Steps 1–2, 5–7; T3 Step 6]

Source report links: [T1](01-Static-Malware-Analysis.md) · [T2](02-GPU-Miner-Detection.md) · [T3](03-Credential-Theft-and-Network-Analysis.md) · [T4](04-Trojan-and-Persistence-Analysis.md)
