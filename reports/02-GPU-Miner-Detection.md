# LuaTools cryptocurrency-mining observation report

> **Public, redacted edition.** Local paths, host-specific identifiers, exact session/file times, network addresses and incidental environment details have been replaced or withheld. Relative time labels are local to this document. This is not an original forensic artifact. Supporting raw artifacts named below are **not included**; see [evidence availability](../docs/EVIDENCE.md) and [redaction policy](../docs/REDACTIONS.md). Claims retain their original limits.

[Repository overview](../README.md) · [Master report](05-Master-Security-Investigation-Report.md)


Test date: 6 November 2026. All timestamps retain [timezone withheld] offset (timezone withheld).

**Conclusion: no affirmative evidence of GPU or CPU cryptocurrency mining was found in this observed run. This is a bounded negative finding, not certainty that mining can never occur.**

The test launched the installed LuaTools application and Steam, inspected modules in both and every observed steamwebhelper process, and measured GPU, CPU, and TCP activity. No game was launched by the analyst. Other desktop applications remained running. The observed installed application is the previously identified 1.3.3 build; the original installer is 1.3.2. The pre-launch hashes below identify the actual tested files. No application setting was changed to disable previously identified behavior; this test does not reverse the earlier riskware/privacy assessment.

## Pre-launch baseline

**Verdict: baseline usable, but the whole computer was not GPU-idle.** LuaTools, Steam, and steamwebhelper were absent before launch. integrated graphics graphics and Windows GPU Engine counters were available. Screen capture, Desktop Window Manager, BACKGROUND-01, and Paint already consumed GPU resources; their activity must not be attributed to LuaTools.

Commands: `Get-Process -Name LuaTools,steam,steamwebhelper`; `Get-CimInstance Win32_VideoController`; five baseline `Get-Counter "\GPU Engine(*)\Utilization Percentage"` captures, with three-second sleeps between captures; `Get-NetTCPConnection`; and `Get-Process | Sort-Object CPU -Descending`. Initialization/collection overhead makes the actual baseline spacing longer than three seconds; timestamps are preserved.

| Baseline GPU PID | Process | Average busiest-engine % | Peak busiest-engine % |
|---|---|---|---|
| PROCESS-103 | BACKGROUND-02.exe | 18.8274 | 21.6210 |
| PROCESS-012 | dwm.exe | 13.1481 | 16.8561 |
| PROCESS-128 | BACKGROUND-01.exe | 6.3829 | 7.9587 |
| 11736 | BACKGROUND-03.exe | 4.0482 | 7.2535 |
| PROCESS-135 | BACKGROUND-01.exe | 1.6444 | 4.3975 |
| 4 | System | 0.3575 | 1.4701 |
| PROCESS-003 | csrss.exe | 0.1010 | 0.4439 |
| PROCESS-126 | Openscreen.exe | 0.0496 | 0.0679 |
| PROCESS-069 | explorer.exe | 0.0368 | 0.1126 |
| PROCESS-102 | msedgewebview2.exe | 0.0000 | 0.0000 |
| PROCESS-112 | TabTip.exe | 0.0000 | 0.0000 |
| PROCESS-129 | WhatsApp.Root.exe | 0.0000 | 0.0000 |

Baseline: 5 GPU snapshots, from T+00:00:05 to T+00:00:28. Mining-port baseline hits: 0. See network evidence for all connections.

Baseline CPU values below are accumulated CPU seconds since process start, not current utilization:

| Process | PID | CPU seconds | Working set MiB |
|---|---|---|---|
| BACKGROUND-01 | PROCESS-128 | 1974.5938 | 802.0742 |
| BACKGROUND-02 | PROCESS-103 | 387.5469 | 220.7891 |
| BACKGROUND-05 | PROCESS-052 | 313.7500 | 115.1250 |
| BACKGROUND-04 | PROCESS-174 | 245.6094 | 502.6875 |
| BACKGROUND-04 | PROCESS-159 | 201.6094 | 416.9531 |
| BACKGROUND-01 | PROCESS-135 | 179.6094 | 352.0547 |
| explorer | PROCESS-069 | 107.2188 | 394.4336 |
| RazerAppEngine | PROCESS-110 | 105.0781 | 98.8242 |
| BACKGROUND-04 | PROCESS-141 | 104.0469 | 203.3867 |
| brave | PROCESS-051 | 88.0781 | 363.5664 |
| dptf_helper | PROCESS-039 | 57.4531 | 8.1953 |
| BACKGROUND-01 | PROCESS-133 | 53.4062 | 176.3281 |
| Openscreen | PROCESS-099 | 48.5000 | 168.2070 |
| brave | PROCESS-189 | 48.1562 | 158.4883 |
| brave | PROCESS-171 | 42.1094 | 435.8438 |
| msedgewebview2 | PROCESS-162 | 24.1406 | 167.7031 |
| msedgewebview2 | PROCESS-145 | 20.7031 | 65.7031 |
| BACKGROUND-03 | 11736 | 20.6406 | 235.2109 |
| brave | PROCESS-106 | 20.2812 | 282.8594 |
| codex | PROCESS-130 | 20.1719 | 169.3828 |

## Step 1 — Static mining and GPU-loader strings

**Verdict: no supported mining indicator or requested compute-loader string found.** Commands: `python work\mining_static.py`, which reads the seven files, extracts ASCII and UTF-16LE strings, searches the exact supplied keyword/library/API lists, and saves offset, complete string, surrounding context, and an individual assessment in `static-scan.json`.

| File | SHA-256 | Mining lexical hits | Compute lexical hits |
|---|---|---|---|
| %LOCALAPPDATA%\LuaTools\current\LuaTools.exe | f88aafd626484943d524a6a4e355633dd8e52b8af698816b0a469b73c7495a45 | 0 | 0 |
| %LOCALAPPDATA%\LuaTools\current\LuaTools.dll | b5792220f2944b9e3be94abfcf55d6d50f1040e7aa1b17965f92d777e6725972 | 32 | 0 |
| C:\Program Files (x86)\Steam\winmm.dll | dc1594774d3003f7c82fbcdaac4cc9bbc81d7ee2ab82dd5f47c4f04cc3bd8236 | 0 | 0 |
| C:\Program Files (x86)\Steam\dwmapi.dll | 8e42f465c276c4d0430fb24158724979605b794a414ba4d1803ad66dd289411b | 0 | 0 |
| C:\Program Files (x86)\Steam\OpenSteamTool.dll | 24a6841e2e484d4586cec5344f71b9913b81692bb0e8ff557b472d565f5dfdfe | 5 | 0 |
| C:\Program Files (x86)\Steam\cloud_redirect.dll | e0fa412d1dc41c40aff0f733c5c42dd2c004a8fafce355ae2ef13b5541349996 | 4 | 0 |
| %APPDATA%\LuaToolsGui\cloudredirect\CloudRedirect.exe | 5a15983d6eb435e80d1375f3d013e87f4dd86d3618fb269ef0c64e2a1f1490a8 | 63 | 0 |

False positives explicitly resolved: ethash occurs inside GetHashCode/AssetHash/CryptGetHashParam; pool refers to thread/connection/array pools and UI icons; wallet refers to Fluent UI icon names; nonce is protocol vocabulary. Donate is the depot-key sharing feature identified previously, not cryptocurrency donation. No supported ethash implementation, stratum, xmrig, hashrate, CryptoNight, monero, bitcoin, coinhive, nicehash, or minexmr mining indicator was identified. The exact raw matches remain in the evidence. Constructed, encrypted, downloaded, or transient code can evade string scans.

## Step 2 — GPU compute import tables

**Verdict: no requested GPU compute imports found.** The scan parsed normal and delay-load imports using pefile. GPU-library filters: cuda, opencl, vulkan, nvml, amdocl. Dynamic loaders were recorded separately.

| File | GPU import DLLs | Dynamic loader functions |
|---|---|---|
| LuaTools.exe | 0 | GetProcAddress; LoadLibraryA; LoadLibraryExW |
| LuaTools.dll | 0 | none in native table |
| winmm.dll | 0 | GetProcAddress; LoadLibraryA; LoadLibraryExW; LoadLibraryW |
| dwmapi.dll | 0 | GetProcAddress; LoadLibraryA; LoadLibraryExW |
| OpenSteamTool.dll | 0 | GetProcAddress; LoadLibraryExA; LoadLibraryExW; LoadLibraryW |
| cloud_redirect.dll | 0 | GetProcAddress |
| CloudRedirect.exe | 0 | GetProcAddress; LoadLibraryA; LoadLibraryExW |

No GPU compute imports were found in any of the seven files. This does not rule out dynamic loading, DirectCompute/graphics-shader computation, a downloaded component, or code injected into another process. The native import table of a managed DLL is not a complete map of its managed capabilities.

## Step 3 — LuaTools running without Steam

**Verdict: no requested GPU compute module found in the app-only snapshot.** LuaTools was launched from `%LOCALAPPDATA%\LuaTools\current\LuaTools.exe`, followed by a 45-second wait. The collector enumerated the complete `.Modules` list, not just matching names. Steam was launched only after this snapshot.

| Phase | Process | PID | All modules | GPU name matches | Error |
|---|---|---|---|---|---|
| luatools-only | LuaTools | PROCESS-137 | 235 | 0 | none |

The filter was `cuda|opencl|vulkan|nvml|cudart|amdocl|compute`. No matches is reassuring for conventional loaded compute DLLs. A matching DLL would establish only a graphics/compute library’s presence, not cryptocurrency mining. Normal rendering engines can legitimately load Vulkan or other compute-capable libraries.

## Step 4 — Steam, LuaTools, and Steam subprocess modules

**Verdict: no requested GPU compute module found in the recorded target-process snapshots.** Steam was launched from `C:\Program Files (x86)\Steam\steam.exe`, followed by a 60-second wait. Modules were collected at combined start, approximately every two minutes, and at combined end for LuaTools, Steam, steamwebhelper, and CloudRedirect if present. Full paths and every module are retained in `modules.jsonl`.

| Phase | Process | PID | Total modules | Compute matches | Error |
|---|---|---|---|---|---|
| luatools-only | LuaTools | PROCESS-137 | 235 | 0 | none |
| combined-start | LuaTools | PROCESS-137 | 238 | 0 | none |
| combined-start | steam | PROCESS-076 | 146 | 0 | none |
| combined-start | steamwebhelper | PROCESS-030 | 124 | 0 | none |
| combined-start | steamwebhelper | PROCESS-193 | 55 | 0 | none |
| combined-start | steamwebhelper | PROCESS-196 | 64 | 0 | none |
| combined-start | steamwebhelper | PROCESS-198 | 94 | 0 | none |
| combined-start | steamwebhelper | PROCESS-200 | 51 | 0 | none |
| combined-start | steamwebhelper | PROCESS-201 | 44 | 0 | none |
| combined-start | steamwebhelper | PROCESS-202 | 64 | 0 | none |
| combined-end | LuaTools | PROCESS-137 | 238 | 0 | none |
| combined-end | steam | PROCESS-076 | 146 | 0 | none |
| combined-end | steamwebhelper | PROCESS-030 | 124 | 0 | none |
| combined-end | steamwebhelper | PROCESS-193 | 55 | 0 | none |
| combined-end | steamwebhelper | PROCESS-196 | 64 | 0 | none |
| combined-end | steamwebhelper | PROCESS-198 | 94 | 0 | none |
| combined-end | steamwebhelper | PROCESS-200 | 51 | 0 | none |
| combined-end | steamwebhelper | PROCESS-201 | 44 | 0 | none |
| combined-end | steamwebhelper | PROCESS-202 | 64 | 0 | none |

Across all phases: 64 process/module snapshots, 0 snapshots with compute-filter matches, 0 recorded module-access errors.

Observed relevant DLL paths (each distinct path shown once per host):

| Host process | PID | Relevant loaded DLL paths |
|---|---|---|
| LuaTools | PROCESS-137 | C:\WINDOWS\SYSTEM32\DWMAPI.dll; C:\WINDOWS\SYSTEM32\WINMM.dll |
| steam | PROCESS-076 | C:\Program Files (x86)\Steam\OpenSteamTool.dll; C:\Program Files (x86)\Steam\WINMM.dll; C:\Program Files (x86)\Steam\winmm_real.DLL; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-030 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-193 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-196 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-198 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-200 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-201 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |
| steamwebhelper | PROCESS-202 | C:\WINDOWS\SYSTEM32\WINMM.dll; C:\WINDOWS\SYSTEM32\dwmapi.dll |

This distinguishes an installed DLL from one actually observed in a process. Standard module enumeration does not reliably expose manually mapped/unlinked modules or arbitrary injected executable memory. No memory dump, private-executable-region analysis, or kernel-level tracing was performed.

Additional observed limitation: Task Manager was present (first recorded T+00:01:13, PID PROCESS-157). It was not launched or closed by the analyst. An observer-aware miner could pause in this environment; this run does not test that counterfactual.

The native cloud_redirect.dll was not observed loaded in the enumerated target processes. Its installed bytes were scanned statically, but cloud-feature activation and other optional execution paths were not exercised in this idle test.

## Step 5 — Extended GPU observation

**Verdict: no sustained target-process GPU workload supporting a miner was observed.** The authoritative collector used `Get-Counter "\GPU Engine(*)\Utilization Percentage" -SampleInterval 15 -MaxSamples 41`: the first capture was warm-up, then 40 interval samples. A separate collector also recorded one-second snapshots about every 15 seconds; these supplementary samples are in `gpu.jsonl`, but the table below uses the continuous collector.

Continuous captures: 41 including warm-up; measurement intervals: 40; measured timestamp span: 601.975 seconds. First collector timestamp: T+00:02:40; final: T+00:12:42. Invalid/out-of-range samples excluded: 0.

For each PID and each interval, process GPU % is the maximum of its engine counters (the busiest-engine measure), not a sum across overlapping engines. Averages are over the 40 intervals; absent per-PID engine samples contribute zero and their coverage is recorded. Full per-engine averages and peaks are in `summary.json`; raw per-engine counters are in `gpu-continuous.jsonl`. These percentages are utilization estimates, not wattage or hashrate.

| Process | PID | Average GPU % | Peak GPU % | Intervals >3% | Intervals >10% | Counter coverage |
|---|---|---|---|---|---|---|
| LuaTools.exe | PROCESS-137 | 1.9700 | 2.5920 | 0 | 0 | 40/40 |
| steamwebhelper.exe | PROCESS-198 | 0.0534 | 1.3458 | 0 | 0 | 40/40 |

Target processes that never acquired a GPU counter instance are listed in process/module evidence; absence of a counter is not a directly measured zero. For comparison, the busiest processes across the entire desktop were:

| Process | PID | Average GPU % | Peak GPU % |
|---|---|---|---|
| BACKGROUND-02.exe | PROCESS-103 | 19.1479 | 21.6442 |
| dwm.exe | PROCESS-012 | 6.0198 | 8.0681 |
| BACKGROUND-01.exe | PROCESS-128 | 5.3072 | 7.0938 |
| LuaTools.exe | PROCESS-137 | 1.9700 | 2.5920 |
| BACKGROUND-01.exe | PROCESS-135 | 1.8260 | 3.0898 |
| BACKGROUND-04.exe | PROCESS-141 | 0.7087 | 1.2740 |
| brave.exe | PROCESS-171 | 0.3728 | 3.8964 |
| System | 4 | 0.0578 | 0.3315 |
| Openscreen.exe | PROCESS-126 | 0.0563 | 0.0887 |
| steamwebhelper.exe | PROCESS-198 | 0.0534 | 1.3458 |
| explorer.exe | PROCESS-069 | 0.0402 | 0.1378 |
| csrss.exe | PROCESS-003 | 0.0008 | 0.0207 |

A universal 0–3% cutoff cannot prove no miner, and >10% does not prove one. Rendering, screen capture, and browser UI activity can exceed those values. Here, target PID behavior, baseline, engine records, modules, and network observations were assessed together. Fifteen-second averages can conceal brief bursts; a delayed, throttled, hardware-specific, or observer-aware miner remains possible.

Target PIDs with no GPU counter instance during the continuous window: steam.exe PID PROCESS-076, steamwebhelper.exe PID PROCESS-030, steamwebhelper.exe PID PROCESS-193, steamwebhelper.exe PID PROCESS-196, steamwebhelper.exe PID PROCESS-200, steamwebhelper.exe PID PROCESS-201, steamwebhelper.exe PID PROCESS-202. Their measured GPU average/peak is **not available**, rather than an asserted numeric zero.

Supplementary one-second snapshot peaks (distinct from the 15-second interval peaks above):

| Process | PID | Peak GPU % |
|---|---|---|
| LuaTools.exe | PROCESS-137 | 5.1497 |
| steamwebhelper.exe | PROCESS-198 | 1.5275 |

LuaTools briefly exceeded 3% in these shorter snapshots, without sustained high utilization in the continuous intervals. UI interaction was not logged, so an exact interaction-to-spike attribution is not claimed.

## Step 6 — Extended TCP connection observation

**Verdict: no evidence of mining-pool traffic from the requested port check.** `Get-NetTCPConnection` was sampled every 20 seconds in the combined window, with baseline/start/end captures as well. All established TCP connections were retained, plus connections with remote ports in the requested set: 3333, 4444, 5555, 9999, 14444, 45560, 1080, 3032, 4545, 5000, 7777, 9000, 9100, 14433, 45700, 8008, 9200.

Combined scheduled polls: 30. Total network snapshots including other phases: 34. Requested-port hits across all recorded phases: 0. Raw connection records include owner PID and timestamp. There is no DNS/reputation attribution or packet-content inspection.

Distinct nonstandard remote TCP endpoints (ports other than 80/443/53; IPv4/IPv6 loopback excluded):

| Owner | PID | Remote address | Remote port | Recorded occurrences |
|---|---|---|---|---|
| BACKGROUND-01.exe | PROCESS-136 | REMOTE-IP-07 | 2096 | 34 |
| BACKGROUND-01.exe | PROCESS-136 | REMOTE-IP-08 | 2053 | 34 |
| steam.exe | PROCESS-076 | REMOTE-IP-04 | 27019 | 32 |

A port number does not establish a mining protocol; ordinary Steam and other applications can use nonstandard ports. Mining can use HTTPS/443 or arbitrary ports, proxies, UDP, and short connections between polls. These checks cannot exclude such traffic or determine the content of encrypted connections. Nonstandard does not imply unencrypted.

## Step 7 — CPU corroboration

**Verdict: no sustained high target CPU workload supporting a CPU miner was observed.** Process CPU seconds and RAM were sampled approximately every 30 seconds throughout the combined period, exceeding the requested five-minute minimum. CPU% was calculated correctly as `100 × delta(CPU seconds) / delta(wall seconds) / logical processor count`, with start-time/PID identity checks. The supplied `.CPU` value alone would not be a percentage.

Exact processor count withheld; CPU normalization used the recorded count. The averages below are weighted by elapsed interval time; peaks are the highest normalized interval averages. RAM is peak working set. Samples and all-process CPU summaries are included for investigating unrelated processes.

| Process | PID | Intervals | Average CPU % | Peak CPU % | Peak RAM MiB |
|---|---|---|---|---|---|
| LuaTools | PROCESS-137 | 20 | 1.3589 | 1.8368 | 484.7031 |
| steamwebhelper | PROCESS-030 | 20 | 0.1683 | 0.4877 | 178.8828 |
| steamwebhelper | PROCESS-202 | 20 | 0.0926 | 0.9255 | 543.8164 |
| steamwebhelper | PROCESS-198 | 20 | 0.0871 | 1.2181 | 175.6836 |
| steam | PROCESS-076 | 20 | 0.0848 | 0.1832 | 114.4453 |
| steamwebhelper | PROCESS-196 | 20 | 0.0437 | 0.1970 | 151.4844 |
| steamwebhelper | PROCESS-200 | 20 | 0.0058 | 0.0358 | 35.3555 |
| steamwebhelper | PROCESS-201 | 20 | 0.0010 | 0.0066 | 24.0039 |
| steamwebhelper | PROCESS-193 | 20 | 0.0006 | 0.0067 | 25.8672 |

A GPU miner need not have high CPU utilization. Conversely, UI rendering, downloads, updates, antivirus scans, and the monitoring tools can cause CPU spikes without mining. No CPU-derived hashrate was inferred.

## Step 8 — GPU power measurement

**Verdict: not measured; explicit limitation.** The GPU is integrated graphics. No nvidia-smi command, accessible HWiNFO/GPU-Z/Intel Power Gadget/Afterburner installation in the checked standard folders, or running sensor tool was found. Discovery results are retained in `power-discovery.json`. Windows powercfg is not an instantaneous per-GPU wattage meter; no wattage was fabricated from it. Baseline GPU watts: unavailable. Combined-run watts: unavailable. Delta: unavailable.

## Timing and evidence integrity

| Time | Event |
|---|---|
| T+00:00:00 | Baseline started; no target launch yet. |
| T+00:00:28 | Baseline complete. Launching LuaTools. |
| T+00:00:28 | LuaTools launch requested. |
| T+00:01:17 | LuaTools-only snapshot complete after 45-second wait. Launching Steam. |
| T+00:01:17 | Steam launch requested. |
| T+00:02:22 | Ten-minute combined observation started. |
| T+00:12:27 | Observation complete. GPU samples=40; network polls=30; CPU samples=20; elapsed seconds=604.7842564. |

Collector scripts, exact static hits, raw counters, process ancestry/path snapshots, full module lists, TCP snapshots, CPU samples, and summaries accompany this report. PID reuse findings are recorded in summary.json. Target hashes are checked again after observation, because these applications can self-update. The analyst did not terminate other desktop processes or alter LuaTools settings. No application binary or memory dump is included in the evidence archive.

End-of-test integrity: 7 of 7 target binaries retained their pre-launch SHA-256. Exact before/after hashes were recorded in the private post-test-hashes.json artifact. LuaTools and Steam were left running after the requested observation.

## Final assessment

No affirmative GPU or CPU cryptocurrency-mining evidence was observed in this test: the seven static targets had no requested compute-library imports/strings, live target module snapshots had no matching compute libraries, the measured target GPU/CPU activity did not show a sustained mining workload, and the requested mining-port checks found no connections. The evidence supports “no mining observed under these conditions,” not “certainly impossible.” Remaining gaps include unavailable power telemetry, uninspected executable memory, encrypted/arbitrary-port or short-lived traffic, deferred/network-delivered code, and behavior triggered outside this ten-minute interval. The prior findings of unsigned Steam modification components and default-enabled plaintext depot-key sharing remain separate security concerns.
