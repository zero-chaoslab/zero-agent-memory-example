# zero-agent-memory-example

Example workspace for [zero-chaoslab/zero-agent-memory](https://github.com/zero-chaoslab/zero-agent-memory),
including the upstream package as a submodule under `extra/zero-agent-memory`
and an agent-facing `skills/` directory made of symlinks.

This repository can also be used as a dedicated memory root. In that setup,
the memory workspace and the project workspaces are intentionally split:
`.zero-memory/`, `AGENTS.md`, and `skills/` stay in this repo, while the agent
operates on one or more separate project repositories. That lets the same
memory graph and task-context history survive as you move between different
projects.

## Memory Root And Project Workspaces

Use this repo as the stable workspace when you want one memory system to serve
multiple projects:

- Start the agent from this memory root when you want project switching to share
  the same recall surface. The agent can still read, edit, test, and run
  commands in another project path, but task handoff and reusable memory stay in
  this root workspace.
- Keep durable context, daily learning, curated memory, observability data, and
  temporary agent scratch data under this repo's `.zero-memory/` directory.
- Keep application or research projects in separate working trees, sibling
  directories, nested checkouts, or submodules, depending on how you prefer to
  organize code.

In practice, this means you can work on Project A today, Project B tomorrow,
and still keep the same `.zero-memory/memory/` graph for reusable workflow
rules, debugging patterns, project facts, and cross-project operating habits.

## File Layout

```text
.
├── .gitmodules
├── .zero-memory/
│   ├── context/
│   ├── daily/
│   ├── memory/
│   ├── observability/
│   ├── skills/                            # generated when memory extracts a stable workflow skill
│   └── tmp/
├── AGENTS.md
├── README.md
├── extra/
│   └── zero-agent-memory/                 # git submodule
│       ├── AGENTS.md
│       ├── README.md
│       ├── install.sh
│       └── skills/
│           ├── zero-context-compact/
│           ├── zero-context-persistence/
│           ├── zero-context-todo-list/
│           ├── zero-memory-curator/
│           ├── zero-memory-reflection/
│           └── zero-memory-visual/
└── skills/                                # symlink surface for agents
    ├── zero-context-compact -> ../extra/zero-agent-memory/skills/zero-context-compact/
    ├── zero-context-persistence -> ../extra/zero-agent-memory/skills/zero-context-persistence/
    ├── zero-context-todo-list -> ../extra/zero-agent-memory/skills/zero-context-todo-list/
    ├── zero-memory-curator -> ../extra/zero-agent-memory/skills/zero-memory-curator/
    ├── zero-memory-reflection -> ../extra/zero-agent-memory/skills/zero-memory-reflection/
    └── zero-memory-visual -> ../extra/zero-agent-memory/skills/zero-memory-visual/
```

- `AGENTS.md`: workspace instructions for agents using zero-memory workflows.
- `extra/zero-agent-memory/`: Git submodule containing the source package, installer, docs, and canonical zero-memory skill implementations.
- `skills/`: local agent-facing skill surface. The `zero-*` entries are symlinks into `extra/zero-agent-memory/skills/`, so updating the submodule updates the canonical skill files without copying them.
- `.zero-memory/context/`: durable task contexts and references for restart-safe work.
- `.zero-memory/daily/`: raw daily learning entries that can be promoted into memory.
- `.zero-memory/memory/`: curated graph-backed memory packages and indexes. This stores reusable knowledge for recall across tasks, such as decisions, debugging patterns, workflow rules, project facts, and skill or workflow routing hints.
- `.zero-memory/observability/`: recall and memory-use event logs plus reports.
- `.zero-memory/skills/`: optional memory-managed skill home generated when memory/reflection shows a reusable workflow deserves a stable skill document. These skills are discovered through `.zero-memory/memory/` routing, opened only when relevant, and can be corrected or refreshed when memory observes better usage during normal user-agent work. They are different from agent-native skill directories such as `.codex/skills`, `.cursor/skills`, or `.claude/skills`, whose skill descriptions may be loaded directly by the agent runtime.
- `.zero-memory/tmp/`: disposable scratch output.
- External or nested project workspaces: optional code checkouts the agent can
  operate on while this repo remains the memory root.

## Bootstrap This Layout

From a fresh repository root, the workspace layout above can be created with:

```bash
git submodule add https://github.com/zero-chaoslab/zero-agent-memory.git extra/zero-agent-memory
cat extra/zero-agent-memory/AGENTS.md >> AGENTS.md

mkdir -p skills .codex .cursor .claude
for skill_path in extra/zero-agent-memory/skills/zero-*; do
  ln -s "../$skill_path" "skills/$(basename "$skill_path")"
done

ln -s ../skills .codex/skills
ln -s ../skills .cursor/skills
ln -s ../skills .claude/skills
[ -f .claude/settings.json ] || cp extra/zero-agent-memory/.claude/settings.json .claude/settings.json
```

Run those commands once when setting up a new memory-root workspace. In an
already configured workspace, review existing `AGENTS.md`, submodule, and
symlink state before repeating them.

## Third-Party Skill Layout

This workspace layout is meant to scale to other skill packs:

1. Put third-party skill packs under `extra/`, preferably as Git submodules.
2. Expose the skills agents should load through top-level symlinks in `skills/`.
3. On first use of a third-party skill from `extra/`, `.zero-memory/memory/` records the choice so future recall can route agents back to the skill. Continued reflection can then refine when to use the skill and how to use it well.
4. When repeated use produces a stable local workflow, memory/reflection can generate, correct, or refresh a focused adapter under `.zero-memory/skills/` and wire it back through `.zero-memory/memory/`.

That keeps third-party code updateable in one place while preserving a stable
agent-facing `skills/` directory and a memory graph that improves its skill
selection guidance over time. You can keep many third-party skill packs under
`extra/` without loading all of their descriptions into the agent context at
once, avoiding attention dilution while still making the right skill discoverable
when memory recall says it is relevant.
