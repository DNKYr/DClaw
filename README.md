# DClaw

This is a learning project: I'm building a small coding agent from scratch to understand how agents like SWE-agent and Claude Code work inside. It's a ReAct loop on top of an OpenAI-compatible API (I use DeepSeek). You give it a prompt, often a GitHub issue, and it reads, edits and runs code in a workspace until it decides it's done.

Nothing here is meant to be production-ready. Expect rough edges and half-built features.

## Inspiration

Most of the ideas come from these papers and codebases:

- **[ReAct](https://arxiv.org/abs/2210.03629)**: the core loop of reasoning, calling a tool, reading the result and repeating (`agent/runner.py`).
- **[MemGPT](https://arxiv.org/abs/2310.08560)**: core memory that the agent edits itself. `MEMORY.md` and `USER.md` are loaded into the system prompt, and the agent updates them with memory tools.
- **[SWE-agent](https://arxiv.org/abs/2405.15793)**: pointing the agent at a GitHub issue and giving it file and shell tools that suit coding work.
- **[Nanobot](https://github.com/HKUDS/nanobot)**: a small, readable agent codebase, and the idea of running compaction concurrently with the main session.
- **[Pi coding agent](https://github.com/earendil-works/pi)**: the edit tool design. `edit_file` first looks for an exact match of `old_string`, then falls back to a fuzzy match that ignores Unicode differences, case and whitespace at the start and end of lines (`agent/tools/edit_util.py`). Also the idea of the session owning the context manager, and when to trigger compaction.

## What works so far

- The async ReAct loop. Parallel tool calls in one response run concurrently.
- Tools: `execute_bash`, `read_file`, `write_file`, `edit_file`, `list_files`, `find_file`, plus `read_memory`, `write_memory` and `edit_memory`. Any `Tool` subclass in `agent/tools/` is picked up automatically.
- Multi-turn sessions. Keep typing prompts, and `/quit` exits.
- Memory in `~/.dclaw/` that carries over between sessions. Writes to `USER.md` are printed as a diff so I can see what the agent thinks it learned about me.
- Logs of each run and session in `~/.dclaw/logs/`.

## What I'm working on

Context compaction. Once the conversation gets close to the model's context window, the older history gets summarized. My design notes are in `plan/context-compaction-design.md`. The code in `agent/context.py` is only partly written for now.

## Running it

You need Python 3.13+ and [uv](https://github.com/astral-sh/uv). There's also a Nix flake if you use direnv.

```bash
uv sync
```

Put your keys in a `.env` file:

```bash
DEEPSEEK_API_KEY=sk-...
GITHUB_ISSUE_URL=https://github.com/<owner>/<repo>/issues/<number>
WORKSPACE=/path/to/workspace
```

Then run one of these:

```bash
uv run main.py              # work on the GitHub issue (the repo needs to be in the workspace already)
uv run test_multi_round.py  # chat with the agent interactively
uv run test_memory.py       # interactive, with the memory tools on
```

A few quirks:
- The workspace path is hard-coded in `prompt.py`.
- Any model you use needs an entry in `model.jsonl` that gives its context window.

## Layout

```
agent/
  runner.py    # the ReAct loop
  session.py   # multi-turn loop
  context.py   # builds the system prompt, tracks context usage
  memory.py    # MEMORY.md / USER.md
  provider.py  # OpenAI-compatible client
  tools/       # tools + auto-discovery
plan/          # design notes
```
