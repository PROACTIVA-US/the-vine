# The Vine - Return Session Prompt

> **Copy this entire file as your first message to Claude when starting a session**

---

## Context

You are building **The Vine** - a product ideation tool where AI agents collaborate to generate and refine product ideas. This is part of the Wildvine ecosystem.

### Repository Location
```
~/Projects/Wildvine/the-vine/
```

### Related Repos
- Meta repo: `~/Projects/Wildvine/`
- Governance docs: `~/Projects/Wildvine/shared/governance/`
- CommandCentral (peer): `~/Projects/CommandCentral/`

---

## What The Vine Is

**Steered agent collaboration for product ideation.**

Multiple AI agents discuss, research, and refine ideas together - with human steering. It's NOT debate. Agents **generate and refine product concepts**.

### The Vine vs AI Arena

| AI Arena (CommandCentral) | The Vine |
|---------------------------|----------|
| Agents **debate** a topic | Agents **generate and refine ideas** |
| Produces consensus | Produces product concepts |
| Fixed rounds | Flexible steering |
| Internal knowledge only | External research (web, YouTube, X) |
| Validates existing ideas | Creates new ideas |

---

## The Vine Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           THE VINE PIPELINE                              │
│                                                                          │
│   [Seed Idea]                                                            │
│        │                                                                 │
│        ▼                                                                 │
│   [Load Personas] ─── Different agent perspectives                      │
│        │                                                                 │
│        ▼                                                                 │
│   [Agent Discussion] ─── Multi-round ideation                           │
│        │                   ├── Generate variations                       │
│        │                   ├── Challenge assumptions                     │
│        │                   └── Build on each other                       │
│        ▼                                                                 │
│   [External Research] ─── Web, YouTube, X, KnowledgeBeast               │
│        │                                                                 │
│        ▼                                                                 │
│   [Synthesis] ─── Combine insights into refined concept                 │
│        │                                                                 │
│        ▼                                                                 │
│   [Human Steering] ─── User guides direction                            │
│        │                                                                 │
│        ▼                                                                 │
│   [Knowledge Capture] ─── Insights preserved to KB                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Critical Requirements

### 1. No Silent Workarounds (Intent Enforcement)

**READ:** `~/Projects/Wildvine/shared/governance/intent-enforcement.md`

- Default failure policy: `FAIL_LOUD`
- Pre-flight check required before running pipeline
- If user configures 4 agents, all 4 must participate
- Never silently skip or substitute agents
- Report failures clearly with actionable options

### 2. Universal Pipeline Model

**READ:** `~/Projects/Wildvine/shared/governance/pipeline-architecture.md`

The Vine is a pipeline. Use the standard model:

```python
from shared.pipelines import Pipeline, Stage, FailurePolicy

vine_pipeline = Pipeline(
    id="the-vine-ideation",
    name="Product Ideation via Agent Discussion",
    stages=[
        Stage("seed", "Seed Idea Input", required=True),
        Stage("personas", "Load Agent Personas", required=True),
        Stage("discuss", "Multi-Agent Discussion", required=True,
              config={"rounds": 3, "required_agents": ["all"]}),
        Stage("research", "External Research", required=False,
              executors=["web-search", "youtube", "twitter"]),
        Stage("synthesize", "Synthesis + Refinement", required=True),
        Stage("human-steer", "Human Steering Input", required=False),
        Stage("capture", "Knowledge Capture", required=True),
    ],
    config=PipelineConfig(
        pre_flight_check=True,
        require_pre_flight_pass=True,
        on_required_missing=FailurePolicy.FAIL_LOUD,
        self_configuring=True,
        knowledge_sources=["product-patterns", "market-research"]
    )
)
```

### 3. Knowledge Capture

**READ:** `~/Projects/Wildvine/shared/governance/repo-agent-design.md`

Every session must automatically:
- Extract entities, decisions, patterns
- Store as memory claims with provenance
- Index to KnowledgeBeast
- Never lose insights from discussions

### 4. Skills as Knowledge

**READ:** `~/Projects/Wildvine/shared/governance/skills-as-knowledge.md`

Skills aren't just YAML config. They're indexed knowledge with:
- Semantic descriptions (for discovery)
- Conflict relationships (for safety)
- Combination suggestions (for guidance)

---

## Capabilities to Build

### 1. Agent Collaboration
- Multiple AI agents with different personas
- Multi-round discussion with memory
- Agents build on each other's ideas
- Configurable number of rounds

### 2. External Research
- Web search integration (Firecrawl, Brave)
- YouTube transcript analysis
- X/Twitter thread ingestion
- KnowledgeBeast query (curated knowledge)

### 3. Human Steering
- User can join the conversation
- Redirect discussion mid-stream
- Provide constraints and priorities
- Approve/reject directions

### 4. Synthesis
- Combine multiple agent perspectives
- Extract key insights
- Generate refined product concept
- Create actionable next steps

### 5. Knowledge Capture
- Auto-extract from all messages
- Create memory claims
- Index to KB with provenance
- Feed back to future sessions

---

## Vertical Positioning

The Vine is the core capability. Different verticals can be positioned:

| Vertical | Focus |
|----------|-------|
| The Vine for Biotech | Drug discovery ideation |
| The Vine for Legal | Case strategy development |
| The Vine for Finance | Investment thesis generation |
| The Vine for Enterprise | Product roadmap exploration |

Each vertical can command different pricing.

---

## Technical Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.11+ |
| API | FastAPI |
| Database | SQLite (dev) / PostgreSQL (prod) |
| Vectors | ChromaDB |
| LLM | Anthropic Claude (primary), OpenAI, Grok, Moonshot |
| Web Research | Firecrawl, Brave Search |
| Protocol | MCP for agent communication |

---

## Suggested Directory Structure

```
the-vine/
├── vine/                         # Python package
│   ├── __init__.py
│   ├── cli.py                    # Click CLI
│   ├── config.py                 # Configuration
│   ├── pipeline/                 # Pipeline implementation
│   │   ├── __init__.py
│   │   ├── stages.py             # Stage definitions
│   │   └── executors.py          # Stage executors
│   ├── agents/                   # Agent management
│   │   ├── __init__.py
│   │   ├── personas.py           # Persona definitions
│   │   ├── discussion.py         # Multi-agent discussion
│   │   └── providers.py          # LLM providers
│   ├── research/                 # External research
│   │   ├── __init__.py
│   │   ├── web.py                # Web search
│   │   ├── youtube.py            # YouTube transcripts
│   │   └── twitter.py            # X/Twitter
│   ├── synthesis/                # Synthesis logic
│   │   ├── __init__.py
│   │   └── combiner.py
│   └── capture/                  # Knowledge capture
│       ├── __init__.py
│       └── extractor.py
├── personas/                     # Persona YAML definitions
│   ├── innovator.yaml
│   ├── critic.yaml
│   ├── researcher.yaml
│   └── strategist.yaml
├── tests/
├── pyproject.toml
└── README.md
```

---

## First Session Goals

1. **Set up project structure** - directories, pyproject.toml, basic CLI
2. **Implement Pipeline class** - from shared/governance/pipeline-architecture.md
3. **Create persona definitions** - YAML format for agent personas
4. **Build discussion stage** - Multi-agent round-based discussion
5. **Add pre-flight check** - Test all agents before running

---

## Key Files to Read First

1. `~/Projects/Wildvine/the-vine/README.md` - Product overview
2. `~/Projects/Wildvine/shared/governance/pipeline-architecture.md` - Pipeline model
3. `~/Projects/Wildvine/shared/governance/intent-enforcement.md` - No workarounds
4. `~/Projects/Wildvine/ARCHITECTURE.md` - Ecosystem overview
5. `~/Projects/CommandCentral/docs/architecture/ai-arena-gaps-and-fixes.md` - Learn from Arena mistakes

---

## Session Protocol

1. Read the governance docs in `shared/governance/`
2. Check the existing pipeline architecture
3. Build incrementally - one stage at a time
4. Test pre-flight checks early
5. Ensure knowledge capture is built in from the start

---

## Commands

```bash
# Navigate to The Vine
cd ~/Projects/Wildvine/the-vine

# Set up Python environment
python -m venv .venv
source .venv/bin/activate
pip install -e .

# Run CLI (after built)
vine --help
vine ideate "A tool for collaborative AI brainstorming"
vine status
```

---

## Remember

- **Intent is sacred** - Never silently work around failures
- **Everything is a pipeline** - Use the standard model
- **Capture knowledge** - Never lose insights
- **Pre-flight required** - Test before running
- **Skills are knowledge** - Semantic, discoverable, combinable

---

*"Ideas don't come from individuals. They emerge from the right conversations."*
