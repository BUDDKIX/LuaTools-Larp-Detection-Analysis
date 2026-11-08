# Prompts used and safer repeat testing

[Repository overview](../README.md) · [Methodology](METHODOLOGY.md)

## How the investigation was requested

I used **four separate test prompts, one for each investigation**, followed by **a fifth prompt to compile the four reports into the master report**. The supplied source document is named `LuaTools_Astra_Prompts.txt`. This page summarizes its structure; it is not a verbatim copy or a ready-to-run replacement for it.

| Prompt | Requested work | Resulting report |
|---|---|---|
| 1 — Static malware analysis | Nine numbered steps covering file inventory, signatures, executable structure, imported functions, strings, versions and installation records | [Static analysis](../reports/01-Static-Malware-Analysis.md) |
| 2 — GPU miner detection | Eight numbered steps plus a baseline, covering code indicators, loaded components, GPU/CPU use, connections and a requested power check | [Mining investigation](../reports/02-GPU-Miner-Detection.md) |
| 3 — Credential theft and network analysis | Ten numbered steps covering credential-related code, connections, local interfaces, cloud logs, file access, DNS and traffic totals | [Credentials and networking](../reports/03-Credential-Theft-and-Network-Analysis.md) |
| 4 — Trojan and persistence analysis | Eleven numbered steps covering Steam-loaded components, hooks, automatic launching, protocol handling, filesystem changes, cloud behavior, updates and memory | [Persistence and injection](../reports/04-Trojan-and-Persistence-Analysis.md) |
| 5 — Master report compilation | Combine the four reports, trace claims to evidence, and preserve inconclusive findings; no new test requested | [Master report](../reports/05-Master-Security-Investigation-Report.md) |

The source asks for the prompts to be used in order, with each report saved before the next stage. There are **38 numbered test steps**, plus baseline and additional checks. This is a description of the requests, not a claim that every check succeeded. For example, GPU power data and reliable credential-file read tracing were unavailable. The [plain-English summary](../reports/06-Plain-English-Summary.md) was added later for readers; it is not a fifth investigation.

## Prompts are questions, not evidence

The supplied prompts contain assumptions and some overly strong instructions. The published results must follow the evidence, even when it contradicts the wording of a prompt. In particular:

- A request to determine mining “with certainty” cannot make a short test conclusive.
- A missing web-download marker does not prove SmartScreen never ran.
- Matching export counts do not prove that a replacement library implements every function correctly.
- A version difference does not prove which update or installation mechanism caused it.
- Unchanged file write times do not rule out reads, and traffic volume alone does not prove data theft.
- A private executable memory area is not automatically malicious shellcode.
- Claims about missing antivirus, a guest-only Steam account or unavailable public source need independent evidence. Those facts were not established by the prompt itself.

The source’s “Computer Use” label is also not evidence that a particular skill was invoked. The methodology describes the tools recorded in the investigation. Adapt questions to the actual sample and report unsuccessful or unavailable checks honestly; do not reuse LuaTools-specific assumptions as facts about another program.

## Before trying this yourself

**Please use VirtualBox or another properly isolated test environment when analysing LuaTools or any other unfamiliar executable. Do not repeat these tests on your everyday PC with personal files or accounts.** A virtual machine reduces exposure but is not a guarantee of safety.

Start with a disposable virtual machine, keep its software updated, and take a clean snapshot before introducing the sample. Keep personal accounts, passwords and files out of it. Disable shared folders, shared clipboard, drag-and-drop and unnecessary device passthrough. Oracle documents the risks of host/guest sharing in its [VirtualBox security guide](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/Security.html).

Keep networking disconnected for offline inspection. If a test needs connections, use a deliberately restricted lab network; do not assume default NAT prevents access to your host or local network, and avoid bridging an unknown sample onto your home/work network. See Oracle’s [networking documentation](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html). If you cannot arrange containment, stop before running the sample and seek experienced help.

Virtual machines can change program behavior and GPU availability. A negative mining test inside a VM is not proof that the program behaves the same on a physical computer. Restoring a snapshot does not undo data already sent outside the VM.

**The original investigation used a live desktop, not this recommended isolated setup.** This advice is for future testing; it does not change the historical environment or make the original results more conclusive. Giving an AI assistant full PC access is not a safety measure.
