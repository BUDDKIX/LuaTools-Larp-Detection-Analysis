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

Automation used PowerShell, Python/pefile, 7-Zip, a custom .NET metadata reader and Windows APIs/counters. No VirusTotal upload was used. Public project documentation and registration records were consulted where documented. The creator calls the assistant Astra in the video and directed the investigation rather than personally reversing the binaries. This attribution does not independently establish the underlying model/version. No independent professional validation is claimed.
