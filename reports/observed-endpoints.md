# Every observed target-process TCP endpoint

> **Public, redacted edition.** Local paths, host-specific identifiers, exact session/file times, network addresses and incidental environment details have been replaced or withheld. Relative time labels are local to this document. This is not an original forensic artifact. Supporting raw artifacts named below are **not included**; see [evidence availability](../docs/EVIDENCE.md) and [redaction policy](../docs/REDACTIONS.md). Claims retain their original limits.

[Repository overview](../README.md) · [Master report](05-Master-Security-Investigation-Report.md)


Includes initialization, both timed phases, extension, and loopback. SynSent is an attempted connection, not proof of successful contact. CloseWait can retain a peer-closed connection. DNS candidates are cache correlations, not verified TLS SNI.

| Process | PID | Remote address | Port | DNS cache candidates | RDAP network name | Observed states | Phases |
|---|---|---|---|---|---|---|---|
| LuaTools.exe | PROCESS-026 | REMOTE-IP-18 | 443 | api.github.com | MSFT | CloseWait, Established | lua-initialization, steam-initialization |
| LuaTools.exe | PROCESS-026 | REMOTE-IP-02 | 443 | analytics.lua.tools, lua.tools | CLOUDFLARENET | Established | lua-alone, lua-initialization |
| LuaTools.exe | PROCESS-026 | REMOTE-IP-23 | 443 | db.lua.tools | [REGISTRY-DETAIL-REDACTED] | Established | lua-alone, lua-initialization |
| LuaTools.exe | PROCESS-026 | 127.0.0.1 | 8080 | none | loopback | Established, SynSent | combined, combined-volume-extension, lua-alone, lua-initialization, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | 8.8.4.4 | 443 | none | GOGL | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-01 | 27020 | [STEAM-SERVICE-HOST] | VC-AP | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-01 | 27018 | [STEAM-SERVICE-HOST] | VC-AP | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-03 | 443 | [STEAM-SERVICE-HOST] | registration unresolved | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-06 | 443 | [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net | VALVE-L1 | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-04 | 27032 | [STEAM-SERVICE-HOST] | VALVE-L1 | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-03 | 27037 | [STEAM-SERVICE-HOST] | registration unresolved | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-06 | 27021 | [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net | VALVE-L1 | Established | combined, combined-volume-extension, steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-05 | 27028 | [STEAM-SERVICE-HOST], p2p-maa2.discovery.steamserver.net | VALVE-L1 | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-20 | 443 | api.steampowered.com | AKAMAI | Established | steam-initialization |
| steamwebhelper.exe | PROCESS-154 | 127.0.0.1 | EPHEMERAL-PORT-04 | none | loopback | Established | combined, combined-volume-extension, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | 127.0.0.1 | EPHEMERAL-PORT-05 | none | loopback | Established | combined, combined-volume-extension, steam-initialization |
| LuaTools.exe | PROCESS-026 | REMOTE-IP-13 | 443 | raw.githubusercontent.com, release-assets.githubusercontent.com | US-GITHUB-20170413 | Established | steam-initialization |
| LuaTools.exe | PROCESS-026 | REMOTE-IP-17 | 443 | github.com | MSFT | CloseWait, Established | steam-initialization |
| LuaTools.exe | PROCESS-026 | REMOTE-IP-10 | 443 | analytics.lua.tools, lua.tools | CLOUDFLARENET | Established | steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-12 | 443 | raw.githubusercontent.com, release-assets.githubusercontent.com | US-GITHUB-20170413 | Established | combined, steam-initialization |
| steam.exe | PROCESS-031 | 127.0.0.1 | EPHEMERAL-PORT-06 | none | loopback | Established | combined, combined-volume-extension, steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-19 | 80 | clientconfig.akamai.steamstatic.com, steamconnecttest.com | [REGISTRY-DETAIL-REDACTED] | Established | steam-initialization |
| steam.exe | PROCESS-031 | 127.0.0.1 | EPHEMERAL-PORT-07 | none | loopback | Established | combined, combined-volume-extension, steam-initialization |
| steamwebhelper.exe | PROCESS-180 | 127.0.0.1 | EPHEMERAL-PORT-03 | none | loopback | Established | combined, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | REMOTE-IP-22 | 443 | none | AKAMAI | Established, SynSent | combined, combined-volume-extension, steam-initialization |
| LuaTools.exe | PROCESS-026 | 127.0.0.1 | 6767 | none | loopback | Established | steam-initialization |
| steamwebhelper.exe | PROCESS-154 | REMOTE-IP-21 | 443 | steamcommunity.com | AKAMAI | Established | combined, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | REMOTE-IP-14 | 443 | none | SKYCA-3 | Established | combined, combined-volume-extension, steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-21 | 443 | steamcommunity.com | AKAMAI | Established | steam-initialization |
| steamwebhelper.exe | PROCESS-180 | 127.0.0.1 | EPHEMERAL-PORT-08 | none | loopback | Established | combined, combined-volume-extension, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | REMOTE-IP-15 | 443 | none | SKYCA-3 | Established | combined, combined-volume-extension, steam-initialization |
| steamwebhelper.exe | PROCESS-154 | REMOTE-IP-20 | 443 | api.steampowered.com | AKAMAI | Established | combined, steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-11 | 443 | manifest.luastools.xyz | CLOUDFLARENET | Established | combined, combined-volume-extension, steam-initialization |
| steam.exe | PROCESS-031 | REMOTE-IP-16 | 443 | client-update.fastly.steamstatic.com | SKYCA-3 | Established | combined |
| steamwebhelper.exe | PROCESS-180 | 127.0.0.1 | EPHEMERAL-PORT-02 | none | loopback | Established | combined |
| steamwebhelper.exe | PROCESS-180 | 127.0.0.1 | EPHEMERAL-PORT-01 | none | loopback | Established | combined, combined-volume-extension |
