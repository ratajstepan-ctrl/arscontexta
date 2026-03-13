# Hermes-agent Platform Adapter

This directory documents how Ars Contexta runs on **Hermes-agent** — locally-hosted models
served through [Ollama](https://ollama.com) (e.g. `hermes3:8b-llama3.1-q5_K_M`), accessible via
Open WebUI, LM Studio, or direct API calls.

---

## Tier

**Tier 3 — minimal infrastructure.** The agent reads instructions from a system prompt and executes
file operations. There are no lifecycle hooks, no plugin skill system, and no native subagent
spawning. The full Ars Contexta methodology operates through context file conventions.

## Relationship to generators/

The generation logic lives at `generators/`. This directory provides a distribution-perspective
reference: what does a Hermes-agent user get, and how do shared components translate to
Hermes-agent's environment?

```
platforms/
├── shared/
│   ├── features/     --> generators/features/ (feature blocks)
│   └── templates/    --> reference/templates/ (note type templates)
├── hermes-agent/
│   ├── README.md     This file
│   └── generator.md  SYSTEM_PROMPT.md generation guide
└── claude-code/
    └── ...
```

## What the platform produces

| Output | Hermes-agent |
|--------|-------------|
| Context file | `SYSTEM_PROMPT.md` — paste into Ollama modelfile, Open WebUI system prompt, or prepend to every conversation |
| Skills location | `prompts/` — plain markdown files invoked by pasting the prompt into chat |
| Hooks | None — replaced by context file instructions for orient/work/persist |
| Settings | `ops/config.yaml` — human-editable configuration |
| Identity | Embedded in `SYSTEM_PROMPT.md` |
| Memory bootstrap | `self/` directory loaded manually at session start |

## How to set up

### Option 1: With Claude Code (recommended for initial generation)

If you have Claude Code available, run `/arscontexta:setup` normally. When the setup wizard
detects a `.hermes-agent/` directory in the project root, it generates Hermes-agent artifacts
instead of Claude Code artifacts:

```bash
mkdir .hermes-agent
```

Then run `/arscontexta:setup` in Claude Code.

### Option 2: Manual setup without Claude Code

If you do not have Claude Code, follow the bootstrapping guide below.

1. Create the vault directory and initialize it:

   ```bash
   mkdir my-vault && cd my-vault
   mkdir .hermes-agent         # platform marker
   mkdir notes inbox archive self ops ops/sessions ops/queue ops/observations ops/tensions ops/methodology prompts templates
   git init
   ```

2. Clone the Ars Contexta repository to get access to the generators:

   ```bash
   git clone https://github.com/agenticnotetaking/arscontexta /tmp/arscontexta
   ```

3. Follow `platforms/hermes-agent/generator.md` to compose `SYSTEM_PROMPT.md` manually,
   OR ask any capable LLM to run the generation steps from `skills/setup/SKILL.md` targeting
   the hermes-agent platform.

4. Load `SYSTEM_PROMPT.md` into your Hermes3 environment:
   - **Ollama Modelfile:** Paste contents into a `SYSTEM` block
   - **Open WebUI:** Set as the model's system prompt in workspace settings
   - **LM Studio:** Paste into the system prompt field
   - **Direct API:** Include as the first `{"role":"system","content":"..."}` message

## Platform characteristics

| Aspect | Detail |
|--------|--------|
| Orientation | Manual — agent reads `self/` at conversation start per context file instructions |
| Validation | Instruction-based — context file reminds agent to check schema on every write |
| Processing pipeline | Manual invocation — paste from `prompts/[skill].md` into chat |
| Session persistence | Manual — agent writes session handoff to `ops/sessions/` per instructions |
| Semantic search | Optional via qmd if Ollama tool-use is enabled |

## Operational notes

**Session start:** Load `SYSTEM_PROMPT.md` as the system prompt. Then in your first message,
orient the agent: "Start a new session. Read `self/` and report current state."

**Invoking skills:** Open `prompts/[skill-name].md` and paste its contents as a user message,
optionally appending the file you want to process: "Run the reduce skill on `inbox/article.md`."

**Session end:** Before closing the conversation, ask the agent to persist state: "End session —
write session summary to `ops/sessions/` and update `self/goals.md`."

**Context window budget:** The available context window depends on your Ollama configuration
and model quantization. Check the actual value with `ollama show hermes3:8b-llama3.1-q5_K_M`
and look for the `context length` field. Budget accordingly:
- Orient: ~10% (self/ files)
- Work: ~70% (source material, notes, connections)
- Persist: ~20% (write summaries, update goals)

Split large processing tasks across multiple conversations using ops/queue/ handoffs.
