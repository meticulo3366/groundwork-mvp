# Session notes — Groundwork build history

A running log of what was built, why, and where it lives. For the product itself see
[`README.md`](README.md); for presenting it see [`DEMO-SCRIPT.md`](DEMO-SCRIPT.md).

## What Groundwork is

A fully local, governed agentic AI demo for a Billing & Account support queue, built as the
centerpiece of a Field CTO technical talk. The thesis: production AI agents don't fail because
the model isn't smart enough — they fail because enterprise context, permissions, workflow logic,
evaluation, and observability aren't engineered for production. Groundwork is the proof: a
deterministic governance layer (permission-aware retrieval, injection scanning, policy gates, a
three-tool allowlist, human approval on writes) wrapped around local LLM generation, with every
claim backed by a live, re-runnable eval.

## Build history

**Pilot plan.** Started from a written 5-day pilot plan (`governed-agentic-pilot-plan.md`) —
objectives, audience, architecture, timeline, scorecard, risks, exec readout template — generic to
any governed-agentic-AI pitch.

**Working demo app.** Built as a single-file app (`index.html` + `serve.js`, zero dependencies):
grounded Agent Assist (Option A, read-only), an Agentic Ticket Queue with human approval (Option
B), a Governance Console over 12 seeded knowledge articles, a 24-case gold evaluation harness, and
a full audit trail with JSON export. Initially wired to the Claude API, then switched to fully
local inference via Ollama per request — no data leaves the machine, zero marginal cost.

**Model selection.** Researched and benchmarked local models against the actual pilot prompt
rather than guessing. Landed on `granite4.1:8b` (IBM's RAG/citation-tuned enterprise model, ~5GB)
with `command-r7b` and `qwen3:14b` as researched fallbacks. Rationale and inference settings live
in `PROMPTS.md`, which the server parses directly at startup (`prompt:answer` / `prompt:draft`
fenced blocks) — editing prompts never requires a code change.

**Verification discipline.** Every claim in the docs was checked against the running app before
being written down: live Ollama calls confirmed via `ollama ps` keep-alive timestamps and real
token counts from the model, all mermaid diagrams parsed with mermaid v11 in a real browser before
each push, every README anchor link validated against GitHub's anchor-generation algorithm, and
every formula in the "what the RAG actually is" section cross-checked against the engine source
line by line.

**Rebrand.** Removed all references to the original inspiration brand; renamed the project
Groundwork throughout code, docs, and file names.

**Documentation.** `README.md` — architecture, two annotated flow diagrams (mermaid), an
entity-relationship data model, a precise five-stage retrieval-architecture writeup (what the RAG
actually is: lexical scoring weights, governance partitioning, the confidence-gate math, exact
prompt assembly, citation validation), the eval scorecard, an OWASP/NIST security mapping, and a
table of contents. `PROMPTS.md` — model research and the canonical system prompts. `DEMO-SCRIPT.md`
— the full presenter material: pre-flight checklist, an 8-beat timed script with exact clicks and
spoken lines, a seeded-data cheat sheet, and Q&A ammunition.

**Presenter/audience separation.** The in-app Demo Guide tab (one-click setup per beat) was pulled
out of the visible navigation — it's reachable only via `#guide` in the URL or by double-clicking
the header title. All demo-oriented content was consolidated into `DEMO-SCRIPT.md`; the README
stays pure product documentation.

**Scoped-refusal behavior.** Confirmed and documented that generic/off-scope questions ("what's
the capital of France?") are refused by deterministic gates (scope, ambiguity, confidence) *before*
the model is ever called — proven live via `engineAsk` returning `modelCalled: false` on four
generic probes. Added as an explicit demo beat and Q&A answer: the refusal is the feature, not a
bug, and the audience is invited to try to break it live.

**Companion deck.** A 13-slide Field CTO talk deck was drafted and then updated in place (via the
Claude Design MCP / browser) to reference Groundwork by name, with its real measured eval numbers
replacing the original placeholder anecdotes, and the closing slide/notes pointing at the 12-minute
live demo. A bundled single-file export lives in `docs/index.html` (suitable for GitHub Pages).

## Where everything lives

| What | Where |
|---|---|
| Code + all docs | `C:\Users\mrzek\claude\groundwork\` → [github.com/meticulo3366/groundwork-mvp](https://github.com/meticulo3366/groundwork-mvp) (Apache-2.0) |
| Deck (editable) | Claude Design project `bda30678-4e22-46e4-849f-43c8422e6d4e`, file "Zeke Dean - Field CTO Deck.dc.html" |
| Deck (bundled export) | `docs/index.html` in the repo |
| Local model | Ollama, `granite4.1:8b` active (fallback chain in `serve.js`); pin via `OLLAMA_MODEL` in `.env` (gitignored) |
| Measured eval numbers | 24/24 gold cases, p50 ~1.6s / p95 ~1.9s, $0 marginal cost, zero citation-guard fallbacks — reproducible via the app's own Eval & Scorecard tab |

## Notes for next session

- Deck edits require driving the Claude Design app through the Chrome extension (browser
  automation) — the `DesignSync` MCP tool cannot authorize in this environment (no interactive
  `/design-login`).
- The "honest failure story" Q&A card in the deck still uses a generic past-pilot anecdote, flagged
  as an open item by the design agent — swap in a real failure story if one exists.
- No Anthropic API credentials exist on this machine; the app's fallback path (deterministic
  template engine) is exercised automatically if Ollama is ever down.
