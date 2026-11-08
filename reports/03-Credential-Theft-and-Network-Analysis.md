# LuaTools credential and network investigation

## How to read this report

Read each finding together with its limitations. For **T1–T4**, **§/§§**, source references and finding labels, use the [short reading guide](05-Master-Security-Investigation-Report.md#how-to-read-this-report). “Not observed” is not a guarantee of safety.

> **Public, redacted edition.** Local paths, host-specific identifiers, exact session/file times, network addresses and incidental environment details have been replaced or withheld. Relative time labels are local to this document. This is not an original forensic artifact. Supporting raw artifacts named below are **not included**; see [evidence availability](../docs/EVIDENCE.md) and [redaction policy](../docs/REDACTIONS.md). Claims retain their original limits.

[Repository overview](../README.md) · [Master report](05-Master-Security-Investigation-Report.md)


Date: 6 November 2026; observation timestamps use [timezone withheld] (timezone withheld).

**Credential theft verdict: Unresolved; no affirmative browser-password theft evidence was established. Network verdict: observed traffic is substantially explainable by authentication, analytics, updates, Steam services, and local integration, but the application cannot be certified clean. Previously confirmed depot-key sharing remains a separate data-exposure risk.**

The installed build tested is 1.3.3, not the original 1.3.2 installer. All eight requested binaries were rescanned and hashed. Steam was shut down through its -shutdown command; LuaTools remained in its tray after CloseMainWindow and its specifically identified process was stopped to obtain the requested app-only baseline. No browser credential database contents, authentication tokens, saved passwords, or Steam depot keys were read by the analyst. Applications were launched as authorized.

This is a sampled connection-metadata investigation, not a complete packet capture. PktMon returned “Failed to communicate with the PktMon driver: Access is denied.” No Procmon/Sysmon collector was available in checked locations; no matching Security event 4663 was found in the initial one-minute query. Absence of an audit event does not establish absence of reads. No TLS interception, certificate installation, memory dump, or native API tracing was performed. Registry ownership and PTR queries were run separately after scheduled observation to avoid contaminating the target-process measurements.

## Pre-launch baseline

**Verdict: uncontaminated by running LuaTools/Steam processes at the clean baseline; other applications remained active.** Exact commands are in network_monitor.ps1. Recorded all TCP states including established connections and listeners, all process identities/ancestry, the DNS cache, local UDP endpoint metadata, adapter counters, and credential-file metadata. Initial already-running connection/DNS snapshots are retained separately and are not mislabeled as the clean baseline.

| Baseline measure | Result |
|---|---|
| Time | T+01:06:31 |
| Established TCP rows | 41 |
| Listening TCP rows | 28 |
| Target processes present | none |
| DNS cache entries | 195 |

| Credential-sensitive path | Exists | Bytes | Baseline UTC last write |
|---|---|---|---|
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Cookies | False | — | — |
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Google\Chrome\User Data\Local State | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Default\Login Data | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Default\Network\Cookies | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Local State | True | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] |
| %APPDATA%\Mozilla\Firefox\Profiles | False | — | — |

The modern Network\Cookies locations and browser Local State files were added to the request’s legacy paths. Firefox Profiles was absent. Reading file metadata does not reveal the file contents.

## Step 1 — Credential indicator scan

**Verdict: no confirmed malicious credential-theft hit.** `python work\network_static.py` scanned ASCII and UTF-16LE in all eight installed binaries for every supplied browser, credential API, theft-tool, decrypt, clipboard, and channel term. Each hit includes its exact match, byte offset, full string, and up to 30 characters of surrounding string context in static-scan.json and reviewed-static-hits.json.

| File | SHA-256 | Credential/channel hits |
|---|---|---|
| %LOCALAPPDATA%\LuaTools\current\LuaTools.exe | f88aafd626484943d524a6a4e355633dd8e52b8af698816b0a469b73c7495a45 | 0 |
| %LOCALAPPDATA%\LuaTools\current\LuaTools.dll | b5792220f2944b9e3be94abfcf55d6d50f1040e7aa1b17965f92d777e6725972 | 0 |
| C:\Program Files (x86)\Steam\winmm.dll | dc1594774d3003f7c82fbcdaac4cc9bbc81d7ee2ab82dd5f47c4f04cc3bd8236 | 0 |
| C:\Program Files (x86)\Steam\winmm_real.dll | 308dd7e06a8925e011d1d3decde646502ef12e906eca830dab5be43a7f64660d | 0 |
| C:\Program Files (x86)\Steam\dwmapi.dll | 8e42f465c276c4d0430fb24158724979605b794a414ba4d1803ad66dd289411b | 0 |
| C:\Program Files (x86)\Steam\OpenSteamTool.dll | 24a6841e2e484d4586cec5344f71b9913b81692bb0e8ff557b472d565f5dfdfe | 0 |
| C:\Program Files (x86)\Steam\cloud_redirect.dll | e0fa412d1dc41c40aff0f733c5c42dd2c004a8fafce355ae2ef13b5541349996 | 2 |
| %APPDATA%\LuaToolsGui\cloudredirect\CloudRedirect.exe | 5a15983d6eb435e80d1375f3d013e87f4dd86d3618fb269ef0c64e2a1f1490a8 | 20 |

| File | Term | Offset | Context | Assessment |
|---|---|---|---|---|
| cloud_redirect.dll | Local State | 0x14c859 | s); failing batch to preserve local state | False positive: Cloud-sync log phrase preserve local state, not Chromium Local State file. |
| cloud_redirect.dll | CryptUnprotectData | 0x186b14 | CryptUnprotectData | Dual-use capability: DPAPI import; provider-token storage context exists, but native data flow is not exhaustively proven. |
| CloudRedirect.exe | Local State | 0x24d9b5 | s); failing batch to preserve local state | False positive: Cloud-sync log phrase preserve local state, not Chromium Local State file. |
| CloudRedirect.exe | CryptUnprotectData | 0x289910 | CryptUnprotectData | Dual-use capability: DPAPI import; provider-token storage context exists, but native data flow is not exhaustively proven. |
| CloudRedirect.exe | Cookies | 0x37df32 | Cookies20 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x3844de | Cookies32 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x38b80a | Cookies24 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x3924c6 | Cookies16 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x3961be | Cookies28 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x398999 | Cookies48 | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622bba | ic_fluent_cookies_16_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622bd6 | ic_fluent_cookies_20_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622bf2 | ic_fluent_cookies_24_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622c0e | ic_fluent_cookies_28_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622c2a | ic_fluent_cookies_32_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x622c46 | ic_fluent_cookies_48_filled | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b105d | ic_fluent_cookies_16_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b107a | ic_fluent_cookies_20_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b1097 | ic_fluent_cookies_24_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b10b4 | ic_fluent_cookies_28_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b10d1 | ic_fluent_cookies_32_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |
| CloudRedirect.exe | Cookies | 0x8b10ee | ic_fluent_cookies_48_regular | False positive: Fluent UI Cookies icons/resources, not browser cookie-file collection. |

No requested Login Data/key4.db/logins.json/etc. path, LSASS/credential enumeration/tool indicator, or clipboard-monitoring API name was found in this scan. That does not exclude obfuscated or dynamically constructed behavior. Managed LuaTools contains explicit CopyToClipboard UI operations; that is not proof of clipboard surveillance.

## Step 2 — DPAPI investigation

**Verdict: legitimate own-token use supported; native decrypt behavior not exhaustively resolved.** `pefile` enumerated cloud_redirect.dll imports and printed strings in the requested ±200-byte window around CryptUnprotectData.

| DLL | Function |
|---|---|
| ADVAPI32.dll | CryptAcquireContextW |
| ADVAPI32.dll | CryptReleaseContext |
| ADVAPI32.dll | CryptGetHashParam |
| ADVAPI32.dll | CryptCreateHash |
| ADVAPI32.dll | CryptHashData |
| ADVAPI32.dll | CryptDestroyHash |
| CRYPT32.dll | CryptUnprotectData |
| CRYPT32.dll | CryptProtectData |
| bcrypt.dll | BCryptGenRandom |

Exact nearby import-name strings: `WinHttpSetOption`, `WinHttpSetTimeouts`, `WinHttpOpenRequest`, `WinHttpAddRequestHeaders`, `WinHttpSendRequest`, `WinHttpReceiveResponse`, `WinHttpQueryHeaders`, `WINHTTP.dll`, `CryptProtectData`, `CryptUnprotectData`, `CRYPT32.dll`, `SHGetKnownFolderPath`, `SHELL32.dll`, `CoTaskMemFree`, `ole32.dll`, `Sleep`, `GetTickCount64`, `GetModuleHandleA`, `GetProcAddress`, `GetComputerNameA`, `WaitForSingleObject`.

The nearby bytes belong to import-name data; adjacency is not a call graph and cannot identify the decrypt input. Both CryptProtectData and CryptUnprotectData are imported. This is consistent with token storage, but malware can also import both, so it is not exculpatory by itself. cloud_redirect strings include TokenStorage/DPAPI failures, DpapiTokenStore, Google/Microsoft provider hosts, and S3/R2 credential filenames. CloudRedirect managed TokenFile.ReadJson/WriteJson calls Unprotect/Protect for its provider-token file path. LuaTools AuthService.LoadStored/SaveStored similarly protects its own AppData\LuaToolsGui\auth.dat, with fields for refresh/access tokens and account profile data. These managed call sites were reviewed without decrypting actual user data.

Supporting excerpts and source IL are in managed-context.txt and the earlier full IL evidence. No native CryptUnprotectData invocation or browser-file read was observed with an API tracer; that remains a gap.

## Step 3 — C2/exfiltration strings and hardcoded endpoints

**Verdict: no requested webhook/tunnel/paste-channel string found; numerous extra legitimate-purpose and unresolved-trust endpoints exist.** The same scan covers every requested C2 keyword, including broad frp/wce substrings, and captures URL/domain/IP candidates with offsets and context. Raw candidates remain in static-scan.json. domain-review.json groups all distinct candidates and explanations.

The short allowlist is not a complete specification of a legitimate application. Static references additionally cover db.lua.tools authentication/download history; GitHub/raw/release asset update hosts; ghproxy.net/ghfast.top/gh.ddlc.top proxy fallbacks; manifest.opensteamtool.com and manifest.steam.run; gmrc.wudrm.com manifest retrieval over HTTP; jsdelivr mirrors; Steam APIs/CDNs; Google/Microsoft OAuth and cloud storage; S3/R2; schema and certificate URLs; UI documentation and community BACKGROUND-01 invitations. None was contacted merely to validate the embedded string. IP-shaped versions and schema URIs were separated from network destinations.

Confirmed code-level data sharing from the earlier IL review: the default-enabled DonateKeys setting can POST ID:DecryptionKey pairs extracted from Steam config\config.vdf to http://REMOTE-IP-09/donatekeys/send. The IP being allowlisted does not make plaintext content-key sharing harmless. The feature is described in the app’s UI as community key donation; it is not evidence of account-password theft. This run did not observe that endpoint in sampled target connections and did not capture a POST body, so actual transmission here is not established.

## Step 4 — LuaTools alone, four minutes

**Verdict: extra authentication endpoint observed; no connection metadata proves theft.** After a 30-second initialization period, 12 scheduled snapshots were collected at 20-second targets over four minutes. Initialization was sampled separately at shorter intervals to capture startup traffic. Steam was absent during this phase.

| Measure | Exact result |
|---|---|
| Initialization snapshots | 7 |
| Four-minute scheduled polls | 12 |

LuaTools startup established TCP/443 connections to REMOTE-IP-18 (api.github.com), REMOTE-IP-02 (analytics.lua.tools), and REMOTE-IP-23 (db.lua.tools). It also attempted local 127.0.0.1:8080 before Steam’s CEF endpoint was available. DNS correlations support these host associations but do not reveal URL paths or transferred values. db.lua.tools is outside the request’s expected two-host set; managed code explains authentication/token refresh and download-history functions there.

The analytics managed method constructs an App Launch event with desktop.lua.tools and /launch/<version> and posts JSON to analytics.lua.tools/api/send. The code uses a browser-like User-Agent. This establishes analytics capability and intended fields, not the exact encrypted bytes sent in this run.

## Step 5 — Steam active, four minutes

**Verdict: additional Steam/plugin/update traffic observed; DLL-specific authorship is unresolved.** Steam initialization was observed for at least 60 seconds. Twelve scheduled 20-second polls followed, plus three extra polls to complete the five-minute adapter measurement. Every steamwebhelper subprocess was included alongside Steam and LuaTools.

| Measure | Count |
|---|---|
| Steam initialization snapshots | 10 |
| Combined four-minute polls | 12 |
| Additional volume polls | 3 |

The complete unique endpoint inventory, with process, PID, port, observed states, phase, DNS candidates, and registry name, is in observed-endpoints.md. It includes the expected Steam-owned manifest.luastools.xyz connection and additional GitHub/CDN/authentication/Steam service connections. No assumption is made that Steam should contact only the manifest server: its own services generate substantial independent traffic. A connection owned by steam.exe cannot identify which loaded DLL initiated it. No unmodified-Steam control run was performed.

## Step 6 — Local HTTP bridge

**Verdict: local data/debug surfaces confirmed where responses succeeded; remote exploitation not demonstrated.** Only the three requested read-only GET paths were probed, after scheduled monitoring. Exact status, headers, body, error, and timestamps are retained in bridge-probes.json.

| Probe | HTTP status | Body length | Error |
|---|---|---|---|
| http://127.0.0.1:6767/ | 404 | 37 | HTTP Error 404: Not Found |
| http://127.0.0.1:6767/loaded-apps | 200 | 26 | none |
| http://127.0.0.1:8080/json | 200 | 7046 | none |

Listener ownership is captured in summary.json: port 6767 bound on 127.0.0.1 under PID 4 (HTTP.sys/System); port 8080 bound on 127.0.0.1 under steamwebhelper. This is loopback exposure, not evidence of a LAN-reachable listener. LuaTools managed SetCors emits Access-Control-Allow-Origin: * and GET, POST, OPTIONS. Successful unauthenticated local responses expose only the contents recorded in the probe; browser cross-origin access depends on browser defenses and is not claimed proven. CEF debug listings, when available, are security-sensitive local control surfaces; no debugger command was executed.

http://127.0.0.1:6767/ exact body: `{"success":false,"error":"Not found"}`.

http://127.0.0.1:6767/loaded-apps exact body: `{"success":true,"apps":[]}`.

CEF /json returned 15 page targets, with 15 debugger WebSocket URLs. Exact metadata is retained in the private bridge-probes.json artifact; no debugger attachment was attempted.

## Step 7 — Cloud redirect logs

**Verdict: no cloud_redirect.log evidence available in the searched locations; actual save-file interception unresolved.** The expected Steam-root file and recursive Steam, LuaToolsGui, and CloudRedirect application directories were checked. Search metadata is in log-search-final.json. Valve’s logs/cloud_log.txt is not substituted as proof of a third-party cloud_redirect hook. No game or cloud sync was intentionally activated; absent logs cannot prove the component never accesses saves. Static cloud-save/token strings establish capability, not this session’s observed save filenames and byte counts.

Live module snapshots confirmed Steam loaded OpenSteamTool.dll, the local winmm.dll and winmm_real.dll. No cloud_redirect.dll was observed in the enumerated target modules. Therefore the cloud hook’s native DPAPI/file access path was not dynamically exercised or traced here.

## Step 8 — Post-run credential metadata

**Verdict: metadata comparison cannot establish credential reads or their absence.** Repeated Get-Item metadata checks used the exact baseline paths after LuaTools-only and after the combined phase. No password database was opened or parsed.

| Path | Baseline last write UTC | Post-run last write UTC | Changed | Baseline/post bytes |
|---|---|---|---|---|
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Cookies | missing | missing | False | None/None |
| %LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Google\Chrome\User Data\Local State | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Default\Login Data | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Default\Network\Cookies | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %LOCALAPPDATA%\Microsoft\Edge\User Data\Local State | [PROFILE-METADATA-REDACTED] | [PROFILE-METADATA-REDACTED] | False | [PROFILE-METADATA-REDACTED] |
| %APPDATA%\Mozilla\Firefox\Profiles | missing | missing | False | None/None |

An unchanged LastWriteTime does not rule out reads. A changed timestamp demonstrates a metadata/content change, not necessarily LuaTools access: browsers and other processes can write concurrently. LastAccessTime is subject to OS update policy and is not a reliable per-process read audit. The available tools did not establish which process read these credential files.

## Step 9 — DNS-over-HTTPS

**Verdict: LuaTools DoH capability is confirmed statically; actual LuaTools DoH use was not confirmed in this run.** Managed DohResolver constructs https://1.1.1.1/dns-query?name= and parses responses. No sampled LuaTools-owned connection to the supplied public-resolver IPs was observed. Connections to 8.8.4.4:443 appeared under steamwebhelper, and other resolver-IP observations belonged to BACKGROUND-04 or closed/ownerless rows. TCP/443 to a DNS provider is a DoH candidate, not proof of DNS message contents.

LuaTools-related entries were present in the ordinary Windows DNS cache, including analytics.lua.tools, db.lua.tools, lua.tools and manifest.luastools.xyz. Cache absence would not prove DoH anyway: TTL expiry, caching inside a process, literal-IP connects, other resolver mechanisms, and incomplete sampling are alternatives. Detailed resolver-IP records are in summary.json. This test did not inspect encrypted DNS payloads.

## Step 10 — Five-minute passive volume

**Verdict: host-wide traffic measured; per-process exfiltration volume unresolved.** `Get-NetAdapterStatistics` was recorded before and after the combined five-minute window. The exact deltas are below. Counters cover the entire adapter, including all browsers, BACKGROUND-01, Steam, monitoring-related traffic, and background services. They are not LuaTools/Steam process counters and were not labeled as such.

| Adapter | Seconds | Sent bytes | Received bytes |
|---|---|---|---|
| Ethernet | 301.769 | 0 | 0 |
| vEthernet (BACKGROUND-08 (BACKGROUND-09 firewall)) | 301.769 | 2120 | 0 |
| Ethernet 2 | 301.769 | 0 | 0 |
| vEthernet (Default Switch) | 301.769 | 2120 | 0 |
| Wi-Fi | 301.769 | 37010276 | 30908557 |
| Ethernet 3 | 301.769 | 0 | 0 |

A threshold of 1 MB does not prove exfiltration, nor does less than 50 KB prove benign behavior. Authentication, UI resources, updates and Steam networking may exceed those values. No supported process-level byte meter was available in this non-elevated session; packet-driver access was denied. No sum across multiple adapters is used, avoiding virtual/physical double counting.

## Registry ownership, names, and protocol limits

RDAP registration identifies the registered network holder, not necessarily the application operator or TLS hostname. Cache mappings may contain multiple services sharing an IP. PTR names are hints only. Ports 443/80 suggest HTTPS/HTTP, but payload inspection was unavailable; all listed connections are known to be TCP. Nonstandard Steam ports are not automatically malicious or unencrypted. Registry queries and PTR resolution were conducted after the observation; failed lookups remain unresolved.

| IP | Registry network name | PTR | Source/error |
|---|---|---|---|
| REMOTE-IP-01 | VC-AP | 103-10-124-4.valve.net | registration record retained privately |
| REMOTE-IP-02 | CLOUDFLARENET | none | registration record retained privately |
| REMOTE-IP-03 | unresolved | none | <urlopen error _ssl.c:1064: The handshake operation timed out> |
| REMOTE-IP-04 | VALVE-L1 | none | registration record retained privately |
| REMOTE-IP-05 | VALVE-L1 | none | registration record retained privately |
| REMOTE-IP-06 | VALVE-L1 | none | registration record retained privately |
| REMOTE-IP-10 | CLOUDFLARENET | none | registration record retained privately |
| REMOTE-IP-11 | CLOUDFLARENET | none | registration record retained privately |
| REMOTE-IP-12 | US-GITHUB-20170413 | [GITHUB-CDN-PTR] | registration record retained privately |
| REMOTE-IP-13 | US-GITHUB-20170413 | [GITHUB-CDN-PTR] | registration record retained privately |
| REMOTE-IP-14 | SKYCA-3 | none | registration record retained privately |
| REMOTE-IP-15 | SKYCA-3 | none | registration record retained privately |
| REMOTE-IP-16 | SKYCA-3 | none | registration record retained privately |
| REMOTE-IP-17 | MSFT | none | registration record retained privately |
| REMOTE-IP-18 | MSFT | none | registration record retained privately |
| REMOTE-IP-19 | [REGISTRY-DETAIL-REDACTED] | none | registration record retained privately |
| REMOTE-IP-20 | AKAMAI | [CDN-PTR-REDACTED] | registration record retained privately |
| REMOTE-IP-21 | AKAMAI | [CDN-PTR-REDACTED] | registration record retained privately |
| REMOTE-IP-22 | AKAMAI | [CDN-PTR-REDACTED] | registration record retained privately |
| REMOTE-IP-23 | [REGISTRY-DETAIL-REDACTED] | [REMOTE-PTR-REDACTED] | registration record retained privately |
| 8.8.4.4 | GOGL | dns.google | registration record retained privately |

| Address | Registered network | Registrant names/handles returned by RDAP |
|---|---|---|
| REMOTE-IP-01 | VC-AP | Valve Corporation |
| REMOTE-IP-02 | CLOUDFLARENET | Cloudflare, Inc. |
| REMOTE-IP-03 | unresolved | unresolved |
| REMOTE-IP-04 | VALVE-L1 | [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; Valve Corporation |
| REMOTE-IP-05 | VALVE-L1 | [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; Valve Corporation |
| REMOTE-IP-06 | VALVE-L1 | [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED]; Valve Corporation |
| REMOTE-IP-10 | CLOUDFLARENET | Cloudflare, Inc. |
| REMOTE-IP-11 | CLOUDFLARENET | Cloudflare, Inc. |
| REMOTE-IP-12 | US-GITHUB-20170413 | GitHub, Inc.; ORG-GI58-RIPE; [REGISTRY-DETAIL-REDACTED]; RIPE-NCC-HM-MNT; us-github-1-mnt |
| REMOTE-IP-13 | US-GITHUB-20170413 | GitHub, Inc.; ORG-GI58-RIPE; [REGISTRY-DETAIL-REDACTED]; RIPE-NCC-HM-MNT; us-github-1-mnt |
| REMOTE-IP-14 | SKYCA-3 | Fastly, Inc. |
| REMOTE-IP-15 | SKYCA-3 | Fastly, Inc. |
| REMOTE-IP-16 | SKYCA-3 | Fastly, Inc. |
| REMOTE-IP-17 | MSFT | Microsoft Corporation |
| REMOTE-IP-18 | MSFT | Microsoft Corporation |
| REMOTE-IP-19 | [REGISTRY-DETAIL-REDACTED] | [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED] |
| REMOTE-IP-20 | AKAMAI | Akamai Technologies, Inc. |
| REMOTE-IP-21 | AKAMAI | Akamai Technologies, Inc. |
| REMOTE-IP-22 | AKAMAI | Akamai Technologies, Inc. |
| REMOTE-IP-23 | [REGISTRY-DETAIL-REDACTED] | [REGISTRY-DETAIL-REDACTED]; [REGISTRY-DETAIL-REDACTED] |
| 8.8.4.4 | GOGL | Google LLC |

## Timeline

| Time | Event |
|---|---|
| T+01:06:27 | Resetting prior LuaTools tray process after graceful close did not exit it; Steam shutdown already requested. |
| T+01:06:29 | Clean pre-launch baseline started. |
| T+01:06:35 | LuaTools launch requested; initialization snapshots follow. |
| T+01:07:08 | Four-minute LuaTools-only observation started. |
| T+01:11:09 | LuaTools-only observation complete; launching Steam. |
| T+01:12:10 | Combined observation started: 12 required polls over four minutes; extended to five minutes for adapter volume. |
| T+01:17:12 | Scheduled observation complete; bridge probes and log review are performed separately. |

End-of-test hashes: 8 of 8 binaries unchanged. LuaTools and Steam were left running. Evidence includes local endpoint/debug metadata and should be treated as private.

## Conclusions

**Credential Theft — Unresolved.** No affirmative browser-password theft, LSASS access, credential-tool signature, or browser credential path was established. The matched credential-like strings resolve to UI/cloud-sync false positives and DPAPI capability; reviewed managed DPAPI use protects the applications’ own authentication/provider tokens. Metadata comparisons do not establish whether credentials were read, and native file/API tracing was unavailable. Default-enabled plaintext sharing of Steam depot decryption keys is a distinct confirmed code-level data-sharing risk; actual transmission during this run was not captured.

**Network Behavior — Explainable observed connections, incomplete assurance.** Every unique sampled target TCP endpoint is listed in observed-endpoints.md, including loopback and failed attempts; registration/PTR evidence is provided separately. Traffic included analytics, authentication, GitHub/update resources, Steam services/CDNs, the LuaTools manifest service, and local HTTP/CEF integration. This is broadly consistent with the utility’s documented functions, but process-level connection metadata and TLS conceal payloads and cannot refute exfiltration. The unencrypted depot-key-sharing capability and local bridge/debug surfaces prevent a clean-security certification. Unseen short-lived/UDP traffic, per-process volume, dormant features, native credential reads, and TLS contents remain gaps.
