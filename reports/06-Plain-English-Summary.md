# LuaTools investigation: the plain-English summary

## How to read this report

Read each finding together with its limitations. For **T1–T4**, **§/§§**, source references and finding labels, use the [short reading guide](05-Master-Security-Investigation-Report.md#how-to-read-this-report). “Not observed” is not a guarantee of safety.

[Back to the README](../README.md) · [Full master report](05-Master-Security-Investigation-Report.md)

**Published: 8 November 2026. Based on tests from 6 November 2026.** This explains the master report in simpler language. It adds no new tests or evidence.

## What is the answer?

**The tests did not establish that LuaTools was a virus, trojan or cryptocurrency miner. They also did not prove it safe.** There were real security concerns, and some important questions could not be answered.

The master report calls it **riskware**: software whose features or behavior create security risks, even when there is not enough evidence to call it malware. That is this investigation’s assessment, not an independent laboratory certification.

| Question | What the tests actually tell us |
|---|---|
| Is it secretly mining cryptocurrency? | No mining was observed during the roughly ten-minute monitoring run. That does not rule out behavior at other times or under other conditions. |
| Is it stealing passwords? | Password theft was not demonstrated. The tests could not reliably track every read of password files or inspect encrypted network messages, so this remains unanswered. |
| Is it a virus or trojan? | A hidden malicious payload was not established. Some powerful and risky features were found, but those alone do not prove malicious intent. |
| Is it completely safe? | These tests cannot support that claim. There are confirmed concerns and gaps in the testing. |

## Who did the work?

I performed this analysis for one of [OneShed’s video essays](https://www.youtube.com/@OneShed).

I did the analysis using **ChatGPT Codex with the GPT-6 Astra model**, with **medium-to-high reasoning effort** and **full access to my PC**. I directed the investigation and used the assistant to perform code inspection and automated tests.

This is my account of the setup I used; the collection records do not independently verify the model or reasoning settings. Full access refers to Codex’s access permissions, not Windows administrator privileges: the recorded tests ran without elevation, and some tracing was unavailable.

The work examined parts of the program’s code and file structure, then watched selected behavior while it ran. It was not a complete professional reverse-engineering audit or a reconstruction of every function using tools such as Ghidra or IDA.

## How were the tests requested?

I used **four prompts, one for each of the four investigations**, then a fifth prompt to combine their reports. A prompt is a set of instructions for the assistant, not proof that a test worked or that an assumption was true. Some requested checks could not be completed. See the [prompt-to-report guide](../docs/PROMPTS-AND-SAFE-TESTING.md).

## If you want to try this yourself

**Use VirtualBox or another properly isolated test environment for LuaTools or other unfamiliar files. Do not use your everyday PC or personal accounts.** Keep personal files out, disable sharing with your real computer, and restrict network access. A virtual machine is not a guarantee of safety, and software may behave differently inside it, especially when graphics hardware is involved.

The original tests used a live desktop, so this is advice for future testing, not a claim about the original setup. Read the [safer testing guidance](../docs/PROMPTS-AND-SAFE-TESTING.md#before-trying-this-yourself) before running anything.

## What was actually found?

### 1. Code can share Steam content keys without encryption

The inspected code contains a feature that can send Steam **depot keys** to a server. These keys unlock downloaded game content; they are not the same as your account password. The setting defaults to enabled when no explicit value is set.

The code sends those keys using HTTP, which does not encrypt the transfer. **The code path was confirmed, but an actual upload was not observed during these tests.** Saying “we caught it stealing passwords” would misrepresent this finding.

[Evidence: master report, section 3.1](05-Master-Security-Investigation-Report.md#31-static-malware-analysis)

### 2. LuaTools components run inside Steam, and Steam can start LuaTools

Extra program files were loaded inside Steam. Several were unsigned, meaning they lacked a verified publisher signature. Being unsigned does not, by itself, mean a file is malicious.

Some components can change how Steam behaves. Starting Steam also caused LuaTools to start again in a controlled test. No separate LuaTools startup entry was found in the inspected Windows locations, but Steam’s own startup entry could provide an indirect route to launch it.

These are reasons to understand the integration before using it. They are not proof of a trojan.

[Evidence: master report, section 3.4](05-Master-Security-Investigation-Report.md#34-trojan-and-persistence-analysis)

### 3. Local debugging features and an unexplained memory area need more investigation

A debugging interface on the same computer exposed a list of 15 Steam browser-page targets. A separate local communication interface also responded. Such interfaces deserve scrutiny, but the tests did **not** demonstrate that someone on the internet could control Steam through them.

A small area of Steam’s memory could run code but was not tied to a known loaded file. Its contents and origin were not identified. It might be ordinary runtime or modification code; it was not proven to be malicious code.

[Evidence: master report, sections 3.3–3.4](05-Master-Security-Investigation-Report.md#33-credential-theft-and-network-behavior)

### 4. Updates and optional cloud features leave unanswered questions

Components reference several update sources. The tests did not establish every check used to verify the publisher of replacement files, and did not observe an actual replacement event. This is a question about trust in future downloads, not evidence that a malicious update occurred.

An optional cloud component was installed, but its native part was not seen running in the observed Steam session. These tests therefore do not establish what it would do when activated with games or cloud accounts.

[Evidence: master report, section 3.4](05-Master-Security-Investigation-Report.md#34-trojan-and-persistence-analysis)

## What did the mining test show?

During roughly ten minutes of monitoring, LuaTools averaged about **1.97% graphics-processor use** and **1.36% main-processor use** in the recorded measurements. The inspected code and loaded components did not produce the requested mining indicators, and repeated connection checks found no matches for the selected mining ports.

That supports “no mining observed in this test.” It cannot rule out a miner that waits longer, uses different connections, responds to different hardware, or behaves differently when monitoring tools are open. Task Manager was present during the test, and graphics-card power consumption was not measured.

[Evidence: master report, section 3.2](05-Master-Security-Investigation-Report.md#32-gpu-miner-detection)

## Why could password theft not be settled?

The checked credential files did not show changed write times. But a program can read a file without changing it. The tests did not reliably record every program reading those files.

Encrypted network messages were not opened and examined. Whole-computer upload totals also could not show how much data LuaTools alone sent. Connections to a server, or a large traffic total, are not proof that passwords were stolen.

[Evidence: master report, section 3.3](05-Master-Security-Investigation-Report.md#33-credential-theft-and-network-behavior)

## What environment was tested, and what are the limits?

- One live Windows desktop with integrated graphics, running without administrator privileges. Other applications were active; this was not a clean, isolated malware laboratory.
- The supplied installer said **1.3.2**, but the installed LuaTools interface actually tested was **1.3.3**. The reports distinguish these versions.
- The exact Windows build, antivirus status and Steam account mode were not established in the source reports.
- Not every feature, game, account state or update was exercised. Not all native code was reconstructed, and not every network message or file read was traced.
- These are observations of particular files and conditions on **6 November 2026**. Other builds, later updates and different conditions may behave differently.

[Full limitations: master report, section 7](05-Master-Security-Investigation-Report.md#7-testing-limitations)

## How is this useful to an ordinary reader?

It separates what was actually found from rumors and assumptions. It gives you specific concerns to consider: default key sharing, code running inside Steam, automatic launching, local debugging access and trust in updates. It also shows exactly where the investigation could not reach a conclusion.

If you keep using LuaTools, the master report recommends explicitly disabling key donation and checking that the setting stays off, reviewing updates and optional integrations, and doing further testing in a separate environment. These measures reduce particular risks; they do not certify safety.

If you remove it, start with supported uninstall or disable controls and preserve needed saves and settings. The [removal guide](../docs/REMOVAL.md) explains the known integration files and follow-up checks. Do not delete Windows system files based on similar filenames. Complete removal was not validated by this investigation.

## What can readers verify publicly?

The repository contains the four individual reports, the master report, this summary and supporting documentation. File fingerprints, called **SHA-256 hashes**, help other researchers identify the exact software files examined. A matching fingerprint does not mean a file is safe.

Personal computer details have been removed or generalized. Raw logs, memory captures and other private collection records are not included, so readers cannot independently recheck every original measurement from this repository alone. See [evidence availability](../docs/EVIDENCE.md).

**The fair conclusion is limited but useful: no mining was observed, malware was not established, and specific security risks and unanswered questions remain.**
