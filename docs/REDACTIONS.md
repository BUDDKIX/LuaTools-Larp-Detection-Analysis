# Public redaction policy

[Overview](../README.md)

The goal is to reduce personal disclosure while keeping the security conclusions understandable. This is data minimization, not a guarantee of perfect anonymity against every possible correlation.

| Original detail | Public treatment |
|---|---|
| Windows username, profile/workspace paths | Environment-variable placeholders or redacted local path |
| Recorded external IP addresses | Stable `REMOTE-IP-NN` labels; original mapping withheld |
| Loopback and common public resolver IPs | Retained as generic protocol context; not the analyst’s address |
| Regional service DNS/PTR/registration details | Generalized or withheld where not needed for the finding |
| Process IDs and ephemeral ports | Neutral labels; PID 0/4 may remain as generic OS identities |
| Exact session timestamps and time zone | Relative labels or redaction; each document has its own origin |
| File timestamps/profile sizes | Withheld where they describe user files |
| Runtime memory addresses | Withheld; allocation size/protection retained |
| Hardware model and unrelated applications | Generalized/pseudonymized |
| Junction’s unusual target string | Neutral nonexistent-target placeholder |
| Software SHA-256, versions, results | Retained; identify the software, not an account |

Endpoint labels are deliberately non-resolvable; do not use them in firewall commands or threat feeds. The reports retain analytical distinctions such as live versus static-only, local versus external, HTTP versus expected HTTPS and unresolved payload content.

No private mapping or original archive is bundled. Upload only this public package, not the earlier evidence ZIPs. Check separately shared media: this repository review does not redact browser tabs, audio, screenshots or existing GitHub history.
