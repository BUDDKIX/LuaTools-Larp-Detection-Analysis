# Claim-to-evidence guide

[Overview](../README.md)

| Evidence-supported wording | Source | Avoid saying |
|---|---|---|
| “We inspected managed IL and tested observable behavior.” | T1 §4; T2–T4 | “We fully reconstructed every native DLL.” |
| “No mining was observed in this ten-minute run.” | T2 §§1–8 | “Mining is refuted” or “it cannot be a miner.” |
| “Code enables plaintext depot-key sharing by default.” | T1 §4 | “We caught it uploading my keys.” |
| “Steam automatically started LuaTools in the controlled launch.” | T4 §5 | “There is no startup chain” or “there were no Run keys at all.” |
| “OpenSteamTool and local proxies were loaded in Steam.” | T4 §§1–3 | “Every imported injection API executed maliciously.” |
| “A local bridge and CEF debug listing were accessible.” | T3 §6; T4 §7 | “Anyone on the Internet can control Steam.” |
| “One private executable region needs attribution.” | T4 §11 | “We found malicious shellcode.” |
| “A silent numeric-app-ID URI branch exists.” | T4 §6 | “Any website can silently install arbitrary DLLs.” |
| “Credential theft is unresolved.” | T3 §§1–2, 8 | “Unchanged timestamps prove no passwords were read.” |
| “An update trust boundary exists.” | T4 §10 | “We observed a compromised silent update.” |
| “The native cloud path was inactive during these tests.” | T3 §7; T4 §9 | “We observed it intercepting all save files.” |

These categories are separate: risky capability, documented intended behavior, observed execution, inferred purpose and confirmed malicious behavior are not interchangeable.
