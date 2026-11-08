# LuaTools Larp Detection Analysis

**Checking the claims behind the comments: GPU miner, virus, trojan—or something the evidence does not establish?**

## Why this investigation exists

I undertook this analysis after comments on [OneShed](https://www.youtube.com/@OneShed)’s LuaTools videos accused the software of being a GPU miner or virus, blamed it for computer crashes, and criticized OneShed for introducing it. That is my account of what prompted the investigation; this repository does not archive or independently authenticate those comments.

The aim was to challenge unsupported claims with evidence and debunk them where the tests justified doing so. **“Larp detection” is the informal theme for examining confident claims that go beyond the evidence.** It is not a technical malware classification or a finding that individual commenters lied. People can report genuine problems without knowing their cause.

The reports follow the evidence, including findings that raise concerns about LuaTools. Supporting OneShed does not require calling the software safe, and criticism alone does not establish that it is malware.

## What the claims look like against the evidence

These are paraphrased claim categories, not quotations from identified commenters.

| Claim being examined | What this investigation supports |
|---|---|
| “It is a GPU/CPU miner.” | No mining evidence was observed during the approximately ten-minute run. Delayed, conditional or evasive mining was not ruled out. |
| “It is a virus or trojan.” | A hidden malicious payload or trojan was not established. The reports identify specific security concerns and assess the software as riskware. |
| “It steals passwords.” | Credential theft remains unresolved: reliable file-read tracing and inspection of encrypted messages were unavailable. |
| “It crashed or damaged my PC.” | This investigation did not examine commenters’ computers, crash dumps or hardware failures. It cannot confirm or disprove their individual experiences or attribute their cause. |
| “These results prove it is safe to install.” | They do not. The findings apply to the tested files and conditions, with substantial limitations. |

**The supported rebuttal is that these tests did not substantiate the mining claim or establish a virus/trojan verdict. That is narrower than proving every allegation false.** Read the [plain-English summary](reports/06-Plain-English-Summary.md) or the [master report](reports/05-Master-Security-Investigation-Report.md) for the evidence and unanswered questions.

## Who did the analysis?

I performed this analysis for one of [OneShed](https://www.youtube.com/@OneShed)’s video essays.

I did the analysis using **ChatGPT Codex with the GPT-6 Astra model**, with **medium-to-high reasoning effort** and **full access to my PC**. I directed the investigation and used the assistant to perform code inspection and automated tests.

This is my account of the setup I used; the collection records do not independently verify the model or reasoning settings. Full access refers to Codex’s access permissions, not Windows administrator privileges: the recorded tests ran without elevation, and some tracing was unavailable.

The analysis examined executable structure, strings, imports/exports and managed .NET IL, alongside live behavior. **This is not equivalent to an exhaustive professional reverse-engineering engagement using tools such as Ghidra or IDA and deeper manual investigation.** Tool choice alone also does not guarantee completeness. The reports document the particular evidence obtained, not a definitive answer to every possible behavior.

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
| Public evidence | Five redacted reports, a plain-English summary and a software-hash inventory; no raw forensic logs or binaries |

## For anyone repeating this analysis

**Use VirtualBox or another properly isolated test environment for LuaTools or other unfamiliar files, rather than your everyday PC.** Keep personal files and accounts out of it. A VM is not a safety guarantee, and its settings and network access matter. The original tests used a live desktop; this recommendation does not describe how those tests were run.

I used **four test prompts, one per investigation, plus a fifth prompt to compile the master report**. See [the prompt-to-report guide and safer testing advice](docs/PROMPTS-AND-SAFE-TESTING.md) for the workflow, precautions and limitations.

## Start here

- **New to security analysis?** Start with the [plain-English summary](reports/06-Plain-English-Summary.md).
- **Want the conclusion and its limits?** Read the [master report](reports/05-Master-Security-Investigation-Report.md).
- **Checking the conclusions?** Use the [claim-to-evidence guide](docs/CLAIMS.md).
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
| [06 — Plain-English summary](reports/06-Plain-English-Summary.md) | What does this mean without the technical detail? | An accessible explanation of the master report, with the same uncertainties |

## How to read this report

**T1–T4** refer to the four individual test reports. **§** means a section or step, and **§§** means several: **T3 §§1–2, 8** means “Test 3, Steps 1, 2 and 8.” A semicolon separates references. These show where a claim’s evidence comes from.

See the [reading guide and clickable report key](reports/05-Master-Security-Investigation-Report.md#how-to-read-this-report) for examples, finding labels and privacy placeholders. “Not observed” does not mean “impossible,” and a risk rating is not proof of malware.

## What the analysis covered

The work included PE/import/export analysis, string inspection, installer and .NET bundle extraction, and inspection of managed intermediate language (IL). It also included live process/module, resource, network, filesystem and memory-region observations.

**It did not include complete native-code disassembly, an exhaustive source audit, TLS decryption or attribution of every private executable allocation.** Not every function in every DLL was reconstructed. No computer-use skill is claimed to have been used for these tests. These are AI-assisted results, not an independent laboratory certification.

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

This investigation gives readers specific behaviors to evaluate instead of a yes/no label based on antivirus reputation. It identifies which file contains a risky code path, which modules were actually loaded, which observations were negative, and which questions still need stronger instrumentation. The [hash inventory](docs/SOFTWARE-HASHES.md) lets researchers check whether they are examining the same bytes; matching hashes do not prove safety.

The results also provide focused follow-up questions: trace native credential-file reads, identify the private executable region, examine update authenticity, and test optional cloud behavior in an isolated environment. Findings apply to the tested builds and conditions, not every past or future LuaTools release.

## Important limitations

This was a single live desktop, not a clean isolated malware lab. Other applications remained active; Task Manager was present during the mining observation. No kernel packet capture or per-process credential-read trace was obtained, HTTPS contents were not decrypted, and GPU power was not measured. Adapter traffic totals belong to the whole machine, not LuaTools alone. Exact source-to-binary equivalence, original installation consent and Steam account mode were not established. Silent URI installation, update replacement and the native cloud hook were not actively exercised.

The non-elevated context blocked some tracing, but **read-only Steam memory queries did succeed**. Do not generalize that all memory inspection required administrator rights.

## Privacy and evidence transparency

Only redacted publication copies are included. Computer/user identifiers, personal paths, raw connection addresses, exact local times, credential-profile metadata and incidental app details are withheld or pseudonymized. Software hashes, versions, test durations and material results are retained. Loopback and well-known public resolver addresses are generic protocol context, not the analyst’s address.

References inside reports to JSON, logs, IL listings or scripts describe private source artifacts; those are **not available in this repository**. Readers can inspect the published reasoning but cannot independently reproduce every original measurement from this package alone. See [what is included and withheld](docs/EVIDENCE.md) and [the redaction policy](docs/REDACTIONS.md).

## Corrections and discussion

Evidence-based corrections are welcome. Quote the report and step, identify the exact build/hash, and explain what contradicts the statement. Do not post personal logs, account data, passwords, keys, or unredacted memory/traffic captures. A documented powerful feature is not automatically a trojan; a disclosed feature can still create security risks.

This repository does not distribute LuaTools binaries, stolen credentials, bypass tooling or instructions for unauthorized access. It documents a bounded investigation.
