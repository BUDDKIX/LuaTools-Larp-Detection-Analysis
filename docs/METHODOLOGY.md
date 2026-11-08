# Methodology and environment

[Overview](../README.md) · [Master report](../reports/05-Master-Security-Investigation-Report.md)

The original observations were made on 6 November 2026. The compilation date is 8 November 2026. This public repository preparation is not a retest.

The target processes were AMD64 on Windows, under a medium-integrity/non-elevated user. Integrated graphics were present. Exact OS edition/build, antivirus status and Steam account mode were not established. Other desktop software remained running. No claim of a guest-only account or an antivirus-free machine is supported. Hardware model, time zone and local identifiers are withheld for privacy.

| Category | What was done | Main limitation |
|---|---|---|
| Static | Hash/signature checks, PE sections/entropy/imports/exports, strings, package extraction, managed IL inspection | No exhaustive native disassembly or reproducible binary/source match |
| Mining | App-only modules, then Steam/helper modules; 40 continuous intervals over about ten minutes; 30 network polls; 20 CPU samples | Short window, monitor-aware behavior possible, no power sensor |
| Network | Four-minute app-only and four-minute combined polling; initialization samples; combined phase extended to five minutes for adapter totals | Not a packet capture; TLS and per-process byte totals unavailable |
| Persistence | Fresh Steam-only launch, 90-second comparison, registry/tasks/services/startup and URI/junction checks | Not an exhaustive installation audit or unmodified-Steam control |
| Memory | Read-only region queries and 64 bytes of a module header | No heap dump, region disassembly, allocation history or creator attribution |

CPU utilization was calculated from CPU-time deltas divided by wall time and the recorded logical processor count. Process GPU summaries used the busiest engine rather than summing overlapping engine counters. The 15-second interval peak and the one-second supplementary peak are different measurements.

Automation used PowerShell, Python/pefile, 7-Zip, a custom .NET metadata reader and Windows APIs/counters. No VirusTotal upload was used. Public project documentation and registration records were consulted where documented. I did the analysis using **ChatGPT Codex with the GPT-6 Astra model**, with **medium-to-high reasoning effort** and **full access to my PC**.

This is my account of the setup I used; the collection records do not independently verify the model or reasoning settings. Full access refers to Codex’s access permissions, not Windows administrator privileges: the recorded tests ran without elevation, and some tracing was unavailable. No independent professional validation is claimed.

## Prompt workflow and future testing

The analyst supplied four test prompts, one per investigation, followed by a fifth master-report compilation prompt. These specify requested work, not proof that every check completed. See [the prompt-to-report guide](PROMPTS-AND-SAFE-TESTING.md).

For repeat testing of LuaTools or other unfamiliar programs, use VirtualBox or another properly isolated environment, with personal data excluded and host sharing/networking restricted. The original live-desktop setup is not recommended as a template for handling unknown files. Virtualization can affect behavior and GPU measurements; see the [safer testing guidance](PROMPTS-AND-SAFE-TESTING.md#before-trying-this-yourself).
