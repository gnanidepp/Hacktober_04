# Verification — 9 October 2026

The prototype was built and executed locally on Windows using Python 3.13.5.
It is not claimed to be production-secure.

## Actual results

- Automated suite: **163 passed, 3 skipped, 0 failed** after correcting a Windows
  newline fixture and fixing SQLite connection cleanup. The suite includes
  policy precedence, tool schemas, sandbox traversal, Windows junction escape,
  hard links, approvals/replay/expiry, file changes, credential separation,
  audit persistence/redaction, required-module failures, and model-adapter failures.
  Google AI Studio tests additionally cover Gemma-only configuration, credential
  headers, exact model metadata/version, all three output formats, malformed
  responses, attempted tool calls, and rejection of Gemini/other models.
  The additional Google prompt-mode regression tests verify that a single
  enclosing JSON fence is accepted without relaxing field/type validation;
  prose, multiple blocks, invalid fields, and invalid types are rejected.
- Skipped explicitly: future CapabilityCapsule session permission expiry;
  real Gemma integration because no inference endpoint/key is configured;
  symlink creation because this host does not grant that privilege. The separate
  Windows junction/reparse-point test passed.
- One upstream test-only warning: Starlette deprecates its current httpx
  TestClient integration. It does not affect these passing results.
- `python -m agentguard.demo`: all eight implemented required scenarios and
  approved deletion passed in isolated temporary test mode. Scenario 9 is PENDING.
  Temporary demo files were cleaned up successfully after the SQLite fix.
- Real FastAPI backend started on `127.0.0.1:8000` in explicit mock mode.
- The combined launcher started both the API and Streamlit on loopback.
- The project built and installed successfully as an editable Python package
  with its test dependencies; the installed runtime runs the application.
- Dashboard application test exercised all four tabs, the visible TEST MODE
  banner, allowed listing, a deletion waiting for approval with its file intact,
  human-approved deletion, and a blocked outside-sandbox move with notes intact.
- API health, dashboard health, and dashboard root each returned HTTP 200.
- The initial local real-model `check-model` failed safely because no Ollama
  endpoint is running. The selected configuration was subsequently changed to
  Google AI Studio's native API and exact `gemma-4-26b-a4b-it` model.
  Missing Google keys cause a clear configuration error; no substitute model is used.

## Environment used

Core installed versions: FastAPI 0.143.0, Uvicorn 0.54.0, Streamlit 1.65.0,
httpx 0.28.1, Pydantic 2.14.0, python-dotenv 1.2.4, pytest 9.1.1.
The initial Windows sandbox blocked Python's restricted temporary-directory ACLs
during dependency installation. A task-local installer helper resolved this;
that helper is not part of the delivered application and does not alter the
security rules or filesystem tools.

## Remaining setup and planned features

The requested destination is `D:\AgentGuard`. The app did not initially grant
write access to that folder, so the working source and archive are staged in
this chat's outputs directory. Do not infer that the files are already on D:.

The user selected Google AI Studio, exclusively Gemma, and subsequently saved
the key in their D: project. Live diagnostics confirmed that Gemma returned a
valid assessment inside a Markdown JSON fence; the original strict raw-JSON
parser rejected that presentation. The staged adapter now accepts exactly one
complete enclosing fence only in Google prompt mode, then applies the same
strict assessment schema. Its live metadata/inference verification passed
against the exact `gemma-4-26b-a4b-it` model using the D: configuration.
The separate opt-in real-Gemma integration test also passed: **1 passed**.
That test loaded the corrected staged adapter with the user's D: configuration.
The key was never displayed or copied into the source package.

The D: adapter has not been edited because this chat lacks write access there.
Copy the staged `agentguard/providers.py` into the D: project before restarting.
Gemma-only enforcement, deterministic policies, and all fail-closed decisions
remain unchanged. The unit suite uses test transports; live verification is
reported separately and does not imply the D: installation is already patched.

## Create/copy/edit upgrade

The staged build now includes `create_file`, `copy_file`, and `edit_file`.
Additional tests cover exclusive creation, independent copies including binary
bytes, no overwrites, UTF-8 byte limits, protected paths, edit previews, human
approval, replay prevention, file-change invalidation, failed atomic replacement,
preview confidentiality, and omission of submitted content/diffs from the audit.
Full submitted-content echoes in model explanations are also omitted; model
paraphrases or partial quotes cannot all be identified automatically.

Dashboard application verification passed against an isolated mock backend on
port 8002: create, copy, visible edit diff, file unchanged before approval, and
approved edit with the independent copy left unchanged.

A separate real-Gemma gateway smoke test passed create, copy, edit preview,
and approved edit against `gemma-4-26b-a4b-it`. Only temporary disposable files
were used; no D: sandbox data was changed and the key was not displayed.

`Install-AgentGuard-Update.ps1` was tested against a disposable existing project:
it verified copied file hashes, backed up replaced code, and preserved the
configuration, data, and sandbox fixtures. The D: project still needs the user
to run this installer because this chat does not have write access to D:.

## Dashboard redesign

The staged dashboard has a navy/teal visual system, sidebar navigation, status
and counter cards, activity feed, readable model/risk and enforced-result panels,
a dedicated approval page, searchable sandbox metadata, and guided filename/
content fields with an optional JSON mode. Planned extensions remain visibly
labeled, and test mode remains clearly identified.

Streamlit application verification passed across all five pages, guided creation,
edit preview and approval, unchanged-before-approval behavior, audit filtering,
sandbox search, and invalid-JSON handling. The redesign changes only the dashboard;
backend enforcement and credentials are unchanged. A dashboard-only installer
backs up the existing UI file before copying the staged replacement.

The theme/contrast follow-up removes forced dark backgrounds and global paragraph
colors. Cards and text inherit Streamlit's native theme, and primary-button text
(including nested label elements) is explicitly dark on mint. Browser checks
confirmed immediate native Light/Dark switching, readable dropdown options,
and primary-button colors of rgb(94,234,212) / rgb(5,42,37) in both modes.

## Requested automatic-file permission profile

Configurable approval tools allow the requested profile to set
`AGENTGUARD_DENIED_TOOLS=delete_file` and `AGENTGUARD_APPROVAL_TOOLS=`.
Tests confirm automatic create/copy/read/edit/move after successful checks,
explicit deletion denial, no approval bypass, rejection of shell/download/
script-execution tools, preservation of sandbox/protected-path checks,
fail-closed model failures, and review for uncertain assessments.

Status and dashboard report the actual permission profile. The original default
approval behavior remains configurable; this profile is applied by the dedicated
installer, which preserves credentials and stores private backups under ignored data/.
Execution of programs outside AgentGuard is not controlled by this application.
The permission installer was verified on a disposable project: only the two
requested environment settings changed, the API-key value and sandbox/data
fixtures were preserved, and original configuration/code backups were created.

CapabilityCapsule, LoopBreaker, and Blast-Radius Preview have explicit extension
interfaces but are not implemented. Approval expiry and file fingerprints are
implemented safeguards, not a claim that those future features are complete.

Read `README.md` for installation, architecture, exact commands, configuration,
the judge demo sequence, license sources, and the trust-boundary limitations.
