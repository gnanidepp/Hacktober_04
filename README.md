# AgentGuard

Hacktoberfest Hack Day Bengaluru 2026.

AgentGuard sits between an agent's proposed tool call and the filesystem.
Gemma 4 analyzes intent and risk; a separate deterministic policy engine makes
the execution decision. All tools operate inside `sandbox/`. The selected setup
uses Google AI Studio's hosted Gemma 4. Local Ollama and hosted OpenAI-compatible
Gemma endpoints are also supported. Non-Gemma model configuration is rejected.

**Prototype, not a production security boundary.** See the limitations below.
Test mode is explicit and visibly labeled; it never claims to be Gemma inference.

## Quick start on Windows

Open PowerShell in this project folder. If you extracted it to the requested D:
location, use:

```powershell
cd D:\AgentGuard
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e ".[test]"
Copy-Item .env.example .env
.\.venv\Scripts\python.exe -m agentguard --mock
```

Open <http://127.0.0.1:8501>. The launcher starts both FastAPI and Streamlit,
bound to loopback, and seeds missing disposable sample files. Ctrl+C stops both.
You do not need to activate the virtual environment. Python 3.11+ is required.
If a port is already occupied, the launcher exits without attaching to it.

The `--mock` flag means **deterministic test mode, no real Gemma**. It is useful
for trying the enforcement layer before installing a model. It is not the final
Gemma demonstration. Without the flag, the configured model must pass an exact
availability check and structured inference probe before the backend starts.

## Connect your Google AI Studio key (selected setup)

Google's [hosted Gemma documentation](https://ai.google.dev/gemma/docs/core/gemma_on_gemini_api)
lists `gemma-4-26b-a4b-it` and `gemma-4-31b-it`. This project selects the former.
Although Google's service is named the Gemini API, the configured model is
**Gemma 4**. The application rejects Gemini model IDs and never falls back to one.

Open the local `.env` file and add your key after `AGENTGUARD_API_KEY=`:

```dotenv
AGENTGUARD_PROVIDER=google_ai_studio
AGENTGUARD_MODEL=gemma-4-26b-a4b-it
AGENTGUARD_ENDPOINT=https://generativelanguage.googleapis.com/v1beta
AGENTGUARD_API_KEY=ADD-YOUR-KEY-HERE-LOCALLY
AGENTGUARD_RESPONSE_FORMAT=prompt
```

```powershell
.\.venv\Scripts\python.exe -m agentguard check-model
.\.venv\Scripts\python.exe -m agentguard
```

The native adapter sends the key only in `x-goog-api-key`, verifies exact model
metadata, and calls only that model's `:generateContent` route. Returned
`modelVersion` must exactly match. The analyzer receives no executable tools.
Quota errors, mismatches, truncation, missing candidates, or malformed assessments
block execution. A failed real-model startup does not enter mock mode.

Hosted Gemma structured-output support is not assumed. The selected `prompt`
mode includes the complete JSON schema in the instruction and strictly validates
the entire response. One enclosing Markdown JSON fence is normalized in Google
prompt mode; surrounding prose, multiple blocks, unknown fields, and invalid
types still fail closed. You may explicitly select `json_schema` or `json_object`
if supported by your endpoint; verification must pass and there is no automatic
format fallback. Run `check-model` with your saved local key to verify your
account's model availability and inference; see `VERIFICATION.md` for actual results.

## Alternative: connect real Gemma 4 locally

Install [Ollama for Windows](https://ollama.com/download/windows) yourself and
start its local service. The official library lists
[`gemma4:e2b`](https://ollama.com/library/gemma4:e2b); it is an example tag,
not a claim that it is already installed on this laptop. Hardware requirements
vary with quantization, context, and runtime; see
[Google's memory table](https://ai.google.dev/gemma/docs/core).

```powershell
ollama pull gemma4:e2b
ollama list
```

Set these lines in the local `.env` file:

```dotenv
AGENTGUARD_PROVIDER=ollama
AGENTGUARD_MODEL=gemma4:e2b
AGENTGUARD_ENDPOINT=http://127.0.0.1:11434
AGENTGUARD_API_KEY=
AGENTGUARD_RESPONSE_FORMAT=json_schema
```

Then verify and start:

```powershell
.\.venv\Scripts\python.exe -m agentguard check-model
.\.venv\Scripts\python.exe -m agentguard
```

`check-model` verifies the exact tag via `/api/tags`, checks Gemma metadata via
`/api/show`, and runs a schema-validated assessment via `/api/chat`.
The [Ollama API supports schema-constrained output](https://docs.ollama.com/capabilities/structured-outputs).
No model is silently substituted. A timeout, unavailable model, malformed JSON,
unexpected JSON field, or mismatched returned model causes the action to fail closed.

## Connect a hosted Gemma endpoint

Use a provider that serves the exact Gemma 4 model over an OpenAI-compatible
chat API. A key alone does not specify a model or provider. Obtain its base URL,
exact model identifier, and key from that provider's documentation/account.

```dotenv
AGENTGUARD_PROVIDER=openai_compatible
AGENTGUARD_ENDPOINT=https://YOUR-PROVIDER.example/v1
AGENTGUARD_MODEL=EXACT-GEMMA-4-MODEL-ID
AGENTGUARD_API_KEY=YOUR-KEY-ADDED-LOCALLY
AGENTGUARD_RESPONSE_FORMAT=json_schema
```

The adapter verifies `/models` and then `/chat/completions`. If the provider
does not support JSON Schema, explicitly set `json_object`, or `prompt` if it
does not support either structured format. Prompt mode still validates the entire
response against the same strict schema; there is no automatic format fallback.
Endpoint responses must report the exact configured model identifier. Providers
that silently alias model identifiers need explicit adaptation and verification.

Ollama also documents [OpenAI-compatible local and cloud endpoints](https://docs.ollama.com/api/openai-compatibility).
Its cloud service currently documents no structured-output support; use the
appropriate explicit format and verify before a demo. The selected provider is
Google AI Studio through the native adapter above; other hosted endpoints have
not been verified. Review their pricing, privacy, and terms before changing providers.

Place credentials only in `.env`; it and generated token files are excluded from
Git. API keys are sent only as headers. Local inference keeps prompts local;
hosted inference sends task text, proposed paths, resource metadata, policy
context, and recent action history to that provider. Proposed create/edit text
is part of the proposal sent to Gemma. Existing read-file contents and the edit
diff are not automatically sent; the diff is shown only to the human administrator.
Rotate the hosted key after the hackathon if that is your plan.

## Dashboard and judge demonstration

The current build supports seven filesystem tools: list, read, create, copy,
edit, move, and delete. The tool selector supplies an editable JSON example.
Create/copy never overwrite; edit replaces full UTF-8 content, shows a live diff
to the human, and requires approval. These remain gateway tools; an autonomous
task agent has not yet been connected.

The dashboard has Overview, Agent Playground, Approvals, Audit Log, and Sandbox Explorer.
Sidebar navigation organizes the console; Guided mode provides filename/content
fields, while JSON mode remains available for explicit tool-call demonstrations.
Use the top-right menu's Theme options to switch between Light, Dark, and System.
The custom console styling inherits that theme; action labels remain high contrast.
It identifies the provider, model, startup verification, and explicit test mode.
Audit timestamps use UTC. Counters show each action's latest decision.

Seed missing examples if you previously deleted a sample:

```powershell
.\.venv\Scripts\python.exe -m agentguard seed
```

The seed command creates missing examples only and does not overwrite files.
For real judging, start with verified Gemma and confirm the green real-model
banner. Use a clear task matching each proposed action; semantic denial by the
real model is possible and should be explained as an enforced assessment outcome.

| User goal | Tool | Arguments | Expected enforcement |
| --- | --- | --- | --- |
| List my sandbox files | `list_files` | `{"path":"."}` | Allowed after successful checks |
| Read the sample notes | `read_file` | `{"path":"notes.txt"}` | Notes returned; audit omits content |
| Create new sandbox notes | `create_file` | `{"path":"new.txt","content":"New notes\n"}` | New UTF-8 file; existing destinations denied |
| Copy my sample notes | `copy_file` | `{"source":"notes.txt","destination":"notes-copy.txt"}` | Independent copy; source unchanged; no overwrites |
| Update my sample notes | `edit_file` | `{"path":"notes.txt","content":"Updated notes\n"}` | Full-file replacement preview; approval required; changed targets invalidate approval |
| Move my notes outside the sandbox | `move_file` | `{"source":"notes.txt","destination":"../escape.txt"}` | Denied; notes unchanged |
| Delete the disposable sample | `delete_file` | `{"path":"delete-me.txt"}` | Approval required; file still exists |
| Same deletion, human clicks Approve once | — | Pending action's exact proposal | Checks run again; approved deletion executes |
| Read the protected sample | `read_file` | `{"path":"protected/important.txt"}` | Denied by deterministic policy |
| Run a shell command | `unknown_tool` | `{}` | Unknown tool denied |

After a blocked request, refresh Sandbox Explorer to see unchanged files.
Use the Audit Log to distinguish the advisory model assessment, base policy,
final decision, approval status, and execution status. The approval desk belongs
to the human; the agent credential cannot access it.

For a reproducible isolated demonstration of all initial scenarios:

```powershell
.\.venv\Scripts\python.exe -m agentguard.demo
```

That command explicitly uses mock mode and temporary disposable files. It prints
PASS/PENDING assertions. Scenario 9 (session permission expiry) is PENDING because
CapabilityCapsule is future work. Existing approval TTL and replay prevention
do not claim to implement task-scoped capability sessions.

## Architecture and extension points

```text
agentguard/
  api.py          authenticated FastAPI routes; human/agent credential separation
  gateway.py      validation → assessment → policy → extensions → approval → execution
  providers.py    isolated Google AI Studio/Ollama/OpenAI-compatible Gemma adapters; explicit mock
  policy.py       authoritative allow / deny / require_approval rules
  tools.py        registered sandbox-only list/read/create/copy/edit/move/delete operations
  audit.py        SQLite action snapshots and append-only processing-stage history
  models.py       strict input and security-assessment schemas
  extensions.py   CapabilityProvider / SequenceAnalyzer / ImpactAnalyzer / ApprovalProvider
  auth.py         independent local admin/agent token generation
  dashboard.py    Streamlit human dashboard
  __main__.py     local launch, model verification, and disposable sample seeding
  demo.py         isolated reproducible security scenarios
demo_files/       source copies of disposable sample files
tests/           deterministic security and API tests; opt-in real-model integration
data/            runtime SQLite DB and token files (ignored)
sandbox/         runtime disposable tool resources (ignored)
```

Only the trusted gateway owns the executable registry. The model has no tools,
permission-issuing methods, or policy mutation methods. HTTP agents can propose
calls but cannot directly execute a registry operation or approve a pending action.

Precedence is explicit prohibition → invalid tool/argument/resource → failed
required check → missing mandatory approval → allow. Early invalid/prohibited
requests are denied before inference. All enabled extension checks run inside the
gateway and may only restrict the base decision. An extension exception or invalid
result denies execution. Approvals re-run model, policies, and modules before use.
Audited decisions and UI fields never create execution permission.

Pending authority lives in private memory bound to an isolated proposal and
resource fingerprints. Approvals are one-use and expire after 300 seconds by
default. Resource changes invalidate the request. Restarting invalidates pending
requests; the gateway never reconstructs an approval from SQLite. A gateway lock
serializes tool executions and approvals in the single supported backend process.

Implemented: Gemma-only model configuration/adapters, intent/risk assessments, deterministic policy,
sandbox path validation (including Windows reparse points), hard-link rejection,
exclusive file creation/copy, UTF-8 edits with before/after diff and atomic replace,
destructive-action approval, file-change revalidation, audit persistence, dashboard,
tests and demonstration.

Planned independently: CapabilityCapsule, LoopBreaker, and Blast-Radius Preview.
Inject implementations into `Gateway(..., capability=..., sequence=..., impact=...)`.
Each implements `check(proposal, context) -> ModuleResult`. Context includes recent
session history, resources, and policy restrictions. The best next step is
CapabilityCapsule: bind server-issued permissions to the original task/session,
check scope/expiry/revocation on every proposal and approval, then implement
scenario 9. SequenceAnalyzer is the LoopBreaker seam; ImpactAnalyzer is the
Blast-Radius Preview seam. The current file fingerprint check is a minimal
approval-integrity safeguard, not a full impact preview.

## Tests

```powershell
.\.venv\Scripts\python.exe -m pytest -q
$env:AGENTGUARD_RUN_INTEGRATION = "1"
.\.venv\Scripts\python.exe -m pytest -q -m integration
```

Run the second command only with a configured real model; it refuses mock mode.
Without that environment flag the real-model test is explicitly skipped.
Tests use disposable temporary sandboxes. Windows junction and symlink tests
report a skip if the host cannot create the relevant test resource. See
`VERIFICATION.md` for the actual run results, rather than assumed passes.

## Local API

Run the backend alone with `python -m agentguard api` (or `api --mock`).
FastAPI docs are at <http://127.0.0.1:8000/docs> when it is running.
All data/execution endpoints require `X-Guard-Token`. `/health` is public and
reveals only service readiness. Random separate tokens are generated at
`data/agent.token` and `data/admin.token`, unless explicitly supplied in `.env`.
Do not give the admin token or data-directory access to your agent.

```powershell
$agentToken = (Get-Content .\data\agent.token -Raw).Trim()
$body = @{ session_id="example"; task="Read sample notes"; tool="read_file";
           arguments=@{path="notes.txt"} } | ConvertTo-Json -Depth 4
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/actions `
  -Headers @{"X-Guard-Token"=$agentToken} -ContentType application/json -Body $body
```

`POST /actions` accepts proposals. Admin-only endpoints are `GET /events`,
`GET /events/{id}`, `GET /approvals`, `POST /approvals/{id}` (body `{"approve":true}`),
and `GET /sandbox`. The agent credential cannot approve. `GET /status` returns
safe configuration/verification information and action counts.

## Policy configuration

The optional requested automatic-file profile permits create/copy/read/edit/move
after successful security checks, and explicitly denies deletion:

```dotenv
AGENTGUARD_DENIED_TOOLS=delete_file
AGENTGUARD_APPROVAL_TOOLS=
```

An empty approval list removes tool-mandated review, not model/policy validation.
Uncertainty or requested clarification may still require a human. Model failure,
high-risk evidence, protected paths, and sandbox escapes still deny execution.
AgentGuard has no download, shell, command-execution, or script-running tools;
even if a file contains downloaded code, the gateway cannot execute it.
This profile does not control programs launched directly outside AgentGuard.

The default `AGENTGUARD_APPROVAL_TOOLS=delete_file,move_file,edit_file` retains
the original approval behavior. The dashboard shows the active policy, and
configuration changes require a restart.

Environment variables are documented in `.env.example`. Defaults protect the
`protected/` subtree, require human approval for every move, edit, and deletion, and
deny all operations when the model fails. Set `AGENTGUARD_DENIED_TOOLS` to a
comma-separated tool list and `AGENTGUARD_PROTECTED_PATHS` to protected relative
path prefixes. Configuration changes require restarting the application; restart
invalidates pending approvals. Model mismatch, suspicious assessment, goal
deviation, and high/critical severity deny execution. Uncertainty or requested
additional information require human review. Approval cannot override a denial.

The prototype limits file operations to independent regular files up to 64 KiB;
reads and edits require UTF-8. Create accepts UTF-8 text; copy preserves bytes.
Editing replaces the entire file after a reviewed diff and mandatory approval.
Create/copy refuse existing destinations and require an existing parent directory.
File text and edit diffs are omitted from SQLite; live edit previews are admin-only.
Directory listing is nonrecursive and bounded to 1000
entries; recursive deletion and overwriting move destinations are unsupported.
Paths use `/`, never drive names or absolute paths. Moves use exclusive
destination creation and do not overwrite existing files.

## Limitations and trust boundary

- This is an application gateway, not an OS sandbox for arbitrary Python code.
  An agent with direct disk access, the admin token, or the same user's process
  privileges can bypass it. Keep executable tools in this service and use the
  restricted HTTP agent token from a separate client.
- Path revalidation, file identity/content fingerprints, link rejection, and a
  gateway lock address ordinary escapes and approval drift. They do not provide
  an atomic, race-free filesystem boundary against a hostile concurrent local
  process. Use OS isolation/ACLs and handle-based Windows filesystem operations
  before claiming protection against that threat.
- The Streamlit dashboard is intended for a trusted laptop user on loopback.
  Do not expose ports on a LAN or the internet. It is not a multiuser service.
  Use one backend process/worker. Token files depend on local account security.
- The model can make incorrect assessments and is not immune to prompt injection.
  Local deterministic rules enforce resources/approvals independently, but semantic
  intent detection still depends on model quality.
- SQLite is a local audit, not tamper-proof evidence. File contents are omitted
  and configured keys/tokens plus common credential patterns are redacted. Arbitrary
  secrets typed into task text cannot all be identified; do not submit sensitive data.
  Full submitted write-content echoes in model reasoning are omitted, but arbitrary
  paraphrases or partial quotes cannot all be detected.
- A durable pre-execution event is required before filesystem mutation, but SQLite
  and filesystem changes cannot form one atomic transaction. A crash/storage failure
  after a mutation can leave an uncertain outcome; inspect the disposable sandbox.
- No rate limiting, multiworker approvals, remote identity management, session-scoped
  capabilities, complete sequence detection, or full blast-radius preview yet.
- A hosted provider's model metadata is an assertion from that provider, not a
  cryptographic proof of weights. A real Gemma endpoint must be configured and
  verified before claiming the mandatory real-model demonstration is complete.

## License and provider terms

AgentGuard's code is MIT-licensed; see `LICENSE`. Google identifies Gemma 4's
model license as [Apache 2.0 in its model card](https://ai.google.dev/gemma/docs/core/model_card_4).
Ollama's runtime is [MIT-licensed](https://github.com/ollama/ollama/blob/main/LICENSE).
No weights or runtime binaries are bundled with this project. Retain each
component's notices when distributing those components.

The [Ollama service terms](https://ollama.com/terms) govern use of its hosted
services, separate from the local runtime's source license. Other inference
providers have their own terms, data handling, and model availability. This
project's source is open source; a complete hosted deployment is not automatically
open source merely because this code is. Documentation checked 9 October 2026.

For the selected Google AI Studio service, review
[Google's Gemini API additional terms](https://ai.google.dev/gemini-api/terms).
Its unpaid-service terms permit submitted content and responses to be used for
product improvement and reviewed by people, with regional exceptions. The
hackathon demonstration should use the disposable sample tasks and files.
