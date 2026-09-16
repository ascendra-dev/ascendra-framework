# Ascendra Framework

**AI-Native Software Delivery**

Ascendra Framework is a PO-led software delivery framework that uses AI as an execution partner. A single skilled Product Owner — someone who can already think across business analysis, solution architecture, engineering, and QA — takes a client project from brief to production alongside Claude, using a structured set of slash commands, templates, and standards.

The framework is not a running platform or a SaaS tool. It's a set of documents, templates, and slash commands you run locally with [Claude Code](https://claude.ai/code). AI generates. You review. You approve. The next step starts.

---

## Who This Is For

A skilled delivery professional who can already think across BA, SA, architecture, engineering, and QA — has delivered software end to end, including client-facing discovery, and can evaluate AI-generated artifacts and redirect the AI when output is wrong or shallow.

**This is a force multiplier, not a teaching tool.** It amplifies existing delivery capability — it does not grant capability you don't already have. See `CLAUDE.md`'s "Who This Is For" for the full profile.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- An Anthropic subscription with Claude access
- Git

---

## Getting Started

```bash
# 1. Clone the framework
git clone https://github.com/zakashah/ascendra-framework
cd ascendra-framework

# 2. Open this folder as your Claude Code session root
#    (slash commands only load from the active root directory)

# 3. Create your first project
/init-project
#    Asks for a project code and name, then creates projects/{CODE}/
#    with every required subfolder

# 4. Complete the project brief
/run-intake

# 5. Start discovery
/run-brd-discovery
```

Every subsequent phase — domain knowledge, BRD, epics, screen design, architecture, stories, implementation, release — is its own slash command, reviewed and approved by you before the next one runs. See **Learn the Framework** below for where each phase is documented in full.

---

## Folder Structure

```
ascendra-framework/
├── .claude/commands/       # 33 slash commands — the framework's executable interface
├── conventions/            # Framework-authoring meta-rules
├── decisions/              # Framework-level Architecture Decision Records — see decisions/index.md
├── projects/               # Your project workspace
│   ├── TEMPLATE/           #   Templates for every generated artifact (framework dependency, do not delete)
│   ├── {CODE}/             #   Your private client project workspace (gitignored)
│   └── index.md            #   Project registry
├── reference/              # Fixed, universal content (estimation models, Core/Extension methodology, defect severity)
├── CLAUDE.md               # The complete rulebook
├── LICENSE
├── PROJECT-LIFECYCLE.md    # Operational command workflow — every step, command, gate
├── README.md
└── SDLC.md                 # Conceptual stage map — the twelve delivery stages
```

Full breakdown of every folder and file: `CLAUDE.md`.

---

## Learn the Framework

| Document | What it's for |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | The complete rulebook — philosophy, folder structure, every slash command, ubiquitous language, project registry conventions |
| [`SDLC.md`](SDLC.md) | The conceptual stage map — what happens and why at each of the twelve delivery stages |
| [`PROJECT-LIFECYCLE.md`](PROJECT-LIFECYCLE.md) | The operational reference — every step, the exact command that runs it, the gate that closes it |
| [`practitioner-guide/`](practitioner-guide/README.md) | **What to actually write, how much, and what breaks downstream if you get a judgment call wrong** — the layer above the other three docs. Built phase by phase; start at `00-orientation.md`. |
| [`decisions/index.md`](decisions/index.md) | Framework-level ADRs — why the framework works the way it does |

Start with `CLAUDE.md` for the full picture, then use `SDLC.md` and `PROJECT-LIFECYCLE.md` as your day-to-day reference once you're running real projects — and the practitioner's guide whenever you hit a judgment call none of the three tells you how to make.

---

## Ecosystem

| Project | Description |
|---------|-------------|
| [ascendra-framework](https://github.com/zakashah/ascendra-framework) | This repo — the delivery framework |
| [ascendra-ui](https://github.com/zakashah/ascendra-ui) | Component library used for client project frontends |

---

## License

MIT — see [`LICENSE`](LICENSE).
