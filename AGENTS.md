# Project agent instructions

## Project

GitHub profile README for PaulDotSH (`README.md`), shown on the profile page. Public: changes go live when pushed. Status: maintained.

## Stack and layout

- Markdown with centred HTML blocks, shields.io badges, tables and GitHub stats images.
- `README.md` is the only content file. There is no build step.
- Sections, in order: About Me, Tech Stack & Skills, Featured Projects, GitHub Activity & Stats, Support & Contact.
- The stats images and the badges are remote images; keep their URLs as they are unless asked to change them.

## Coding style and rules

- Git: commit or push only when asked. Never add AI co-author trailers (e.g. `Co-Authored-By: Claude`) to commits.
- Everything in the README is public. Do not add private contact details, secrets or tokens.
- Keep the badge and table style of the existing sections.

## Output-heavy commands

- **Delegate output-heavy commands to a cheap model.** When a command's output
  is large but the answer you need is small (parsing logs, sifting build or CI
  output, scanning long dumps), prefer handing it to a cheap subagent and
  taking back only the conclusion: DeepSeek via the `deepseek` MCP
  (`run_task`) or Claude Haiku. Give it the exact command and what to report;
  it reads, it does not edit. This needs no explicit opt-in.
- **Test runs print failures, not passes.** Run suites in a mode that hides
  passing tests and shows only failures plus the summary: e.g. `cargo test -q`,
  `cargo nextest run --status-level fail`, `pytest -q`, `vitest --reporter=dot`,
  `go test ./...` without `-v`, `dotnet test -v q`. If the runner has no such
  mode, filter its output down to the failures. Never pull hundreds of passing
  lines into context.

## Subagent usage

These rules are always in effect, even when the `dispatching-parallel-agents`
skill is not loaded. If you do dispatch subagents, follow them; the skill has
the full workflow.

- **Explicit opt-in only.** Never dispatch, spawn, or delegate to subagents
  unless the developer explicitly asks for it in the current task (e.g. "use
  subagents", "dispatch these in parallel", "delegate this"). A task looking
  parallelizable is not authorization, and neither is a past instruction. If
  parallel work would clearly help, ask once; absent a yes, do the work yourself,
  sequentially. Exception: delegating an output-heavy command to a cheap model
  (see "Output-heavy commands") needs no opt-in.
- **Cheap models for subagents.** The coordinating session runs the expensive
  model; helpers run a cheap/fast one: ChatGPT Luna, DeepSeek v4.1 Flash, Claude
  Sonnet, or Muse-Spark. Pick the cheapest that can do the job, never silently a
  premium model, and state the chosen model in the dispatch and the summary.
- **Poll at the harness cadence.** Polling too often destroys prompt-cache
  retention. Claude Code: check every 4 minutes. Codex and opencode running
  OpenAI models: check every 25 minutes, otherwise wait for completion, and do
  not poll faster. Manual polling only — no tight auto-poll loops. If a result
  arrives earlier, use it immediately.
