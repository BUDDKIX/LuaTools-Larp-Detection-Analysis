# Is LuaTools a virus? A bounded security investigation

Companion repository for **“I reverse engineered LuaTools to see if it’s a virus.”**

## About the video and who did the analysis

I made this video to investigate claims that LuaTools might be a GPU/CPU miner, malware, a virus, or a trojan. **I directed the investigation; I did not personally reverse engineer the binaries.** The AI assistant I refer to as **Astra** in the video performed the code inspection and automated tests through the Codex desktop environment.

Here, “reverse engineered” refers to examining executable structure, strings, imports/exports and managed .NET IL, alongside live behavior. **This is not equivalent to an exhaustive professional reverse-engineering engagement using tools such as Ghidra or IDA and deeper manual investigation.** Tool choice alone also does not guarantee completeness. The reports document the particular evidence obtained, not a definitive answer to every possible behavior.

The purpose is to test those community claims honestly: what supports them, what was not observed, and what remains unknown. This project does not claim to have proven LuaTools safe—or proven a hidden malicious payload.

**Assessment: riskware with confirmed security concerns. No cryptocurrency mining was observed during the bounded test. Credential theft remains unresolved; a trojan was not established.**

This repository explains the evidence behind that assessment, including negative findings and unanswered questions. It is not a malware-detection product, an endorsement, or a safety certificate.

| Study scope | Recorded value |
|---|---|
| Test date | 6 November 2026 |
| Compilation | 8 November 2026; public edition redacted |
| Supplied installer | LuaTools 1.3.2 |
| GUI actually tested | LuaTools 1.3.3 |
| Plugin record / cloud GUI | v2.2 / CloudRedirect 2.6.6.0 |
| Environment | Live Windows desktop; AMD64 target processes; non-elevated user; integrated graphics |
| Exact OS build / antivirus / Steam account mode | Not established in the source reports |
| Analysis | AI-assisted PowerShell/Python/.NET metadata inspection and Windows API/counter measurements |
| Public evidence | Five redacted reports and a software-hash inventory; no raw forensic logs or binaries |

## Start here

- **Want the conclusion and its limits?** Read the [master report](reports/05-Master-Security-Investigation-Report.md).
- **Checking the video’s claims?** Use the [claim-to-evidence guide](docs/CLAIMS.md).
- **Want the individual investigations?** Choose a report below.
- **Considering keeping or removing the tool?** See the [risk-reduction and removal guide](docs/REMOVAL.md).
- **Want to reproduce or challenge a result?** Read [methodology](docs/METHODOLOGY.md), [evidence availability](docs/EVIDENCE.md), and [contribution guidance](CONTRIBUTING.md).

| Report | Question | Supported conclusion |
|---|---|---|
| [01 — Static analysis](reports/01-Static-Malware-Analysis.md) | What is inside the files? | Suspicious capabilities and data-sharing code; no conclusive malicious-payload verdict |
| [02 — Mining investigation](reports/02-GPU-Miner-Detection.md) | Was mining observable? | No mining evidence observed during the approximately ten-minute run |
| [03 — Credentials and networking](reports/03-Credential-Theft-and-Network-Analysis.md) | What data access/network activity could be established? | Explainable sampled connections; credential reads and encrypted payloads unresolved |
| [04 — Persistence and injection](reports/04-Trojan-and-Persistence-Analysis.md) | How does it load, persist and expose features? | Steam-triggered auto-launch and local modules confirmed; a trojan not established |
| [05 — Master report](reports/05-Master-Security-Investigation-Report.md) | What do the results mean together? | Riskware, with specific confirmed concerns and material testing gaps |

## What “reverse engineered” means in this project

The work included PE/import/export analysis, string inspection, installer and .NET bundle extraction, and inspection of managed intermediate language (IL). It also included live process/module, resource, network, filesystem and memory-region observations.

**It did not include complete native-code disassembly, an exhaustive source audit, TLS decryption or attribution of every private executable allocation.** The video title is not a claim that every function in every DLL was reconstructed. The work was performed through a Codex desktop agent using local tools. “Astra” is the creator’s name for the assistant in the video; the collection records do not independently establish its underlying model/version. No computer-use skill is claimed to have been used for these tests. These are AI-assisted results, not an independent laboratory certification.

## The findings that matter most

| Finding | Evidence status | What it does not establish |
|---|---|---|
| Default-enabled plaintext depot-key sharing in LuaTools.dll | Confirmed in inspected managed code | Actual key transmission during the observed run was not captured |
| Local unsigned proxies and OpenSteamTool inside Steam | Observed in module maps | Loaded code alone does not prove malicious execution |
| Steam starts LuaTools through its proxy loader | Reproduced; loader success count increased by one | No direct LuaTools Run entry was found, but Steam itself had one |
| Multiple component update sources | Code/configuration/documentation support update capability | A replacement event and independent publisher authentication were not established |
| Local HTTP bridge and CEF debugging | Read-only probes succeeded; debug listing exposed 15 page targets | Internet reachability or browser-based exploitation was not demonstrated |
| One 64 KB private execute-read region | Observed in Steam | Its creator/content was not identified; it could be legitimate generated/hook code |
| Optional native cloud redirection | Installed and inspected statically | It was not loaded or exercised in the observed run |

Source steps: T1 §4; T3 §§6–8; T4 §§1–3, 5, 7, 9–11. See the master report for the complete register.

### The depot-key finding, precisely

The inspected `DonateKeysService` can read `DecryptionKey` entries from Steam’s configuration, filter previously donated IDs, and POST ID/key pairs as plain text over HTTP. The current null `DonateKeys` setting resolves to enabled. The UI describes community key donation.

These are **Steam depot content-decryption keys, not demonstrated account passwords**. The concern is the enabled-by-default sharing and unencrypted transport. The code path is confirmed; **actual upload in this test is unobserved**. The public edition pseudonymizes the numerical backend address while preserving the endpoint path and HTTP finding.

## Negative findings are not guarantees

- No requested GPU-compute library names/imports or live module matches were found. LuaTools averaged about **1.97% GPU** and **1.36% CPU** in the measured run. Its peak was **2.59% over 15-second GPU intervals**, with a **5.15% supplementary one-second snapshot**. No requested mining-port hits occurred across 30 scheduled polls.
- No affirmative browser-stealer or requested webhook/tunnel pattern was established. Some lexical hits were ordinary icons, runtime APIs, or cloud-sync messages.
- No RWX sections or non-resource section entropy above 7.2 was found. This does **not** exclude all packing, encryption or malicious code.
- Credential-file write timestamps did not change. This does **not** exclude reads.

## Why this is useful

This investigation gives viewers specific behaviors to evaluate instead of a yes/no label based on antivirus reputation. It identifies which file contains a risky code path, which modules were actually loaded, which observations were negative, and which questions still need stronger instrumentation. The [hash inventory](docs/SOFTWARE-HASHES.md) lets researchers check whether they are examining the same bytes; matching hashes do not prove safety.

The results also provide focused follow-up questions: trace native credential-file reads, identify the private executable region, examine update authenticity, and test optional cloud behavior in an isolated environment. Findings apply to the tested builds and conditions, not every past or future LuaTools release.

## Important limitations

This was a single live desktop, not a clean isolated malware lab. Other applications remained active; Task Manager was present during the mining observation. No kernel packet capture or per-process credential-read trace was obtained, HTTPS contents were not decrypted, and GPU power was not measured. Adapter traffic totals belong to the whole machine, not LuaTools alone. Exact source-to-binary equivalence, original installation consent and Steam account mode were not established. Silent URI installation, update replacement and the native cloud hook were not actively exercised.

The non-elevated context blocked some tracing, but **read-only Steam memory queries did succeed**. Do not generalize that all memory inspection required administrator rights.

## Privacy and evidence transparency

Only redacted publication copies are included. Computer/user identifiers, personal paths, raw connection addresses, exact local times, credential-profile metadata and incidental app details are withheld or pseudonymized. Software hashes, versions, test durations and material results are retained. Loopback and well-known public resolver addresses are generic protocol context, not the creator’s address.

References inside reports to JSON, logs, IL listings or scripts describe private source artifacts; those are **not available in this repository**. Readers can inspect the published reasoning but cannot independently reproduce every original measurement from this package alone. See [what is included and withheld](docs/EVIDENCE.md) and [the redaction policy](docs/REDACTIONS.md).

## Corrections and discussion

Evidence-based corrections are welcome. Quote the report and step, identify the exact build/hash, and explain what contradicts the statement. Do not post personal logs, account data, passwords, keys, or unredacted memory/traffic captures. A documented powerful feature is not automatically a trojan; a disclosed feature can still create security risks.

This repository does not distribute LuaTools binaries, stolen credentials, bypass tooling or instructions for unauthorized access. It documents a bounded investigation.
