---
type: config
status: active
tags: [system, prefix-rules]
---
# File Naming Prefix Rules

## Naming Format

| Location | Format | Example |
|----------|--------|---------|
| `{inbox}/` | `domain-prefix-Title` | `AI-tool-OpenClaw`, `Invest-insight-ThreeQuestions`, `Plan-plan-StockExpertAI` |
| Domain directory | `prefix-Title` | `tool-OpenClaw`, `insight-ThreeQuestions` |
| `{plans}/` (folder) | Natural language | `StockExpertAI/` |
| `{plans}/` (file inside folder) | `prefix-Title` | `plan-StockExpertAI` |

> When distributing: move file from inbox to domain directory, remove the domain prefix from filename.
> Plans special case: create a folder with the Title name first, then place the file inside.

---

## Prefix Definitions (8 prefixes)

### skill
> Reusable technical asset — install-and-use capability modules

- Claude Code Skills, custom scripts, automation modules
- Other people's skill breakdowns / collections
- **Routing**: `skill-` files route to `Skills Library/`

**Boundary**: `skill` = installable/callable module | `tool` = specific tool usage

---

### prompt
> Reusable prompt templates or prompt engineering methods

- Complete prompt template collections
- Prompt design techniques and structures
- **Routing**: `prompt-` files route to `Prompt Library/`

---

### learn
> Learning notes, tutorials, reference materials — digested external input

- Learning roadmaps, step-by-step tutorials
- Course notes / reading notes
- Curated resource collections, tool comparisons, reference docs

**Judgment**: "I learned X, here are my notes"
**Boundary**: `learn` = digested external input | `insight` = your own observations

---

### tool
> Specific tools, software, plugins — and practical tips

- Tool reviews / usage records
- Plugin configuration guides
- Quick operations / workarounds, troubleshooting records

**Judgment**: "Using X, you can achieve Y"
**Boundary**: `tool` = specific software/product/tip | `skill` = complete capability module | `build` = system engineering

---

### framework
> Structured mental models — reusable analysis and decision frameworks

- Mental models / thinking frameworks
- Methodology systems
- Analysis frameworks (e.g. Five Forces, OODA)
- Architecture design (system architecture, layered design)

**Judgment**: "This is a system for thinking about X"
**Boundary**: `framework` = structured, reusable across contexts | `insight` = single observation, not a reusable structure

---

### insight
> Cognitive breakthroughs — perspective-changing observations, cases, reviews

- Industry insights / trend predictions
- Case analysis / breakdowns
- Reviews and reflections
- Others' viewpoints collected

**Judgment**: "I realized X"
**Boundary**: `insight` = single observation/discovery | `framework` = reusable model applicable to different contexts

---

### build
> Engineering implementation — technical details and design artifacts

- Technical proposals / architecture implementations
- System building records
- Code-level practice summaries

**Judgment**: "I'm building X"
**Boundary**: `build` = building something (artifacts/design) | `plan` = intending to do (intent/plan) | `learn` = learning something

---

### plan
> Project plans / proposals — ideas that need to be pushed forward

- New project proposals, feature planning
- Ideas to be implemented
- Routes to: `{plans}/` (create folder, then place file inside)

**Judgment**: "I plan to do X"
**Boundary**: `plan` = has intent to push forward, not started | `build` = designing/implementing, already in progress

---

### UDF
> Undefined — content fetch failed and prefix cannot be determined from available info

- Only used when fetch fails AND title provides insufficient information
- Requires manual confirmation before distribution
- **Do not guess**: only use when genuinely undeterminable

---

## Classification Decision Flow

```
Given a piece of content, ask yourself:

1. Is it an installable/callable module? → skill
2. Is it a prompt template? → prompt
3. Is it externally-sourced learning notes or reference? → learn
4. Is it a specific tool/product/practical tip? → tool
5. Is it a reusable thinking structure? → framework
6. Is it an insight/viewpoint/case study? → insight
7. Is it engineering implementation details? → build
8. Is it a project plan/proposal? → plan
9. Completely undeterminable? → UDF (manual confirmation needed)
```

---

## Domain Prefix Mapping

| Domain Prefix (inbox) | Target Directory |
|-----------------------|-----------------|
| `AI` | `50_AI/` |
| `Invest` | `60_Investment/` |
| `Plan` | `01_Plans/` |

> Customize these mappings to match your vault structure.

---

## Distribution Rules

### Core Principle

Subdirectories are **topic containers**, not type containers. The same subdirectory can hold files with different prefixes. Distribution always starts with **topic judgment**, not prefix.

### Plans Routing

All `plan-` and `build-` files go to `{plans}/`.

**Special operation**: Create a folder with the Title name, then place the file inside.

### AI Routing (example for 50_AI/)

**Layer 1: Prefix lock (2 libraries, no exceptions)**

| Prefix | Target Subdirectory |
|--------|-------------------|
| `skill` | `Skills Library/` |
| `prompt` | `Prompt Library/` |

**Layer 2: Topic judgment (all other prefixes)**

| Topic | Target Subdirectory | Common Prefixes |
|-------|-------------------|-----------------|
| Engineering/building/implementation | `AI Engineering/` | `build` `tool` `learn` `insight` `framework` |
| Architecture/system design/principles | `AI Architecture/` | `framework` `insight` |
| Cognition/trends/viewpoints/cases | `AI Cognition/` | `insight` `framework` |
| Reference materials/tool collections | `AI Resources/` | `tool` `learn` |

> **Judgment order**: About how to build things → Engineering; About how to design systems → Architecture; About understanding AI → Cognition; General references → Resources.

### Investment Routing (example for 60_Investment/)

**All topic-based, prefix does not determine subdirectory.**

| Topic | Target Subdirectory |
|-------|-------------------|
| Macro economy/policy/global landscape | `Macro/` |
| Business models/products/startups | `Business Cognition/` |
| Social/psychology/human nature | `Social Cognition/` |
| Trading techniques/charts/indicators | `Technical/` |
| Crypto-related | `Crypto/` (with sub-topics) |

> Customize all directory names and routing rules to match your vault.
