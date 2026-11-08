# Evidence availability

[Overview](../README.md)

## Included

- Five redacted report copies, with step references and retained measured results.
- A plain-English summary of the master report; this adds no new tests or evidence.
- Software SHA-256 identifiers, copied from the original inventory.
- Scope, limitation, redaction and correction guidance.
- A summary of the supplied prompt document and safer testing guidance; the original prompt file is not reproduced.
- `PUBLIC-FILES.sha256`, which identifies the public files in this release. It is an integrity manifest, not a signature or safety endorsement.

## Withheld

Original JSON/process/module/network snapshots, DNS caches, local file metadata, registry exports, debugger target listings, IL dumps, raw logs, analyst scripts with local paths, binary payloads, memory content and private redaction mappings are not included. No promise is made to release them publicly. Historical filenames mentioned by reports refer to these private artifacts, not missing downloads that readers should expect to find here.

## Consequence

This is a **report-only publication with selected identifiers**, not a complete reproducible forensic dataset. The raw measurements cannot all be independently reprocessed from this repository. The redacted copies are intentionally not byte-identical to the original reports. Software hashes identify analyzed files; they do not prove who compiled a file, that a file is safe, or that all experiments were independently validated.

Public copies preserve software version/hash facts, durations, utilization metrics, module counts, findings, uncertainty and source-step references. Local values are replaced or withheld as described in REDACTIONS.md. No new live measurements were made while preparing this repository.
