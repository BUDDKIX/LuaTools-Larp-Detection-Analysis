# Risk reduction and removal

[Overview](../README.md) · [Original recommendations](../reports/05-Master-Security-Investigation-Report.md#9-recommendations)

These are proposed actions, not actions performed by this publication. Removal of the observed integration is not proof that the entire computer is clean.

## If keeping it

1. Explicitly disable depot-key donation if you do not want that sharing; the tested null setting defaulted to enabled.
2. Review documented update controls and optional components. The observed TOML did not explicitly contain `update.enabled=true`; native defaults were not independently proved.
3. Disable unused Steam integration/debug features through supported controls and verify listeners/markers do not reappear.
4. Investigate unresolved native/cloud behavior in an isolated environment, without introducing unnecessary sensitive data.
5. Establish antivirus status and obtain appropriate file/network tracing before treating the software as verified safe.

## If removing it

1. Back up settings or save data you need. Close Steam, its helpers, LuaTools and CloudRedirect. Prefer a supported uninstall/disable workflow when available; its completeness was not tested.
2. Verify the actual Steam directory and identity of each observed third-party file before removing: `winmm.dll`, `winmm_real.dll`, `dwmapi.dll`, `OpenSteamTool.dll`, `cloud_redirect.dll`, and the additional unsigned `xinput1_4.dll`. Review `opensteamtool.toml`, loader logs and application-owned scripts as residuals. **Never delete the genuine System32/SysWOW64 DLLs or unrelated Steam/game files.**
3. If the `.cef-enable-remote-debugging` entry is still the confirmed junction, remove only the link. In Command Prompt, use `rmdir "<VERIFIED-STEAM-DIRECTORY>\.cef-enable-remote-debugging"` after replacing the placeholder. Do not add `/s`, traverse the target, or delete the target directory. Stop if the object type/path differs from expectations.
4. Review/remove the confirmed `HKEY_CURRENT_USER\Software\Classes\luatools` URI key after exporting it if rollback is needed. No additional direct LuaTools Run/task/service entry was established. Do not remove the genuine Steam Client Service; disabling Steam’s own login entry is a separate preference.
5. After preserving needed data, remove residual `%LOCALAPPDATA%\LuaTools`, `%APPDATA%\LuaToolsGui` and `%APPDATA%\CloudRedirect` folders. They may contain settings/provider authentication data. This is not a proven exhaustive inventory of every possible residual.
6. Launch Steam and verify LuaTools no longer starts, the identified local proxies/OpenSteamTool/cloud DLL are absent from its module map, and the URI registration/debug marker/listeners are no longer recreated by this integration. Use supported Steam repair/reinstallation if needed; do not download replacement Windows DLLs from arbitrary websites.
