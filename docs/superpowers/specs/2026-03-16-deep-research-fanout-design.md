# Deep Research Fan-Out System

## Purpose

A self-contained Claude Code workspace that converts any research request into 15-25 parallel subagents, each producing a standalone markdown artifact. Designed for burning unused weekly limits on high-value reference material.

## Supported Research Modes

| Mode | Trigger examples | Decomposition strategy |
|---|---|---|
| Open topic | "deep research on MEV landscape on Solana" | Facets: history, current state, key players, technical deep-dives, trends, risks, opportunities, comparisons, contrarian takes |
| GitHub repo | "research this repo: https://github.com/org/repo" | Architecture, entry points, dependencies, security surface, testing, build system, contribution patterns, code quality, key abstractions |
| Codebase problem | "investigate why auth service leaks connections, codebase at ~/projects/myapp" | Symptoms, hypotheses, related code paths, dependency analysis, similar OSS issues, potential fixes, testing strategies, monitoring gaps |
| Competitor intel | "competitor analysis on Cursor vs Windsurf vs Claude Code" | Product features, tech stack, pricing, positioning, recent changes, user sentiment, hiring signals, funding/runway, strengths/weaknesses |

## Architecture

The system consists of a single `CLAUDE.md` file and a folder convention. No scripts, no dependencies.

### CLAUDE.md Responsibilities

1. Detect research intent from natural language
2. Classify into one of the four research modes
3. Decompose the topic into 15-25 independent research angles
4. Dispatch all angles as parallel `Task` subagents (`general-purpose`, `run_in_background: true`)
5. Monitor completion via `Read` on output files
6. Generate `_index.md` summary after all agents finish

### Output Structure

```
research/
  YYYY-MM-DD/
    <topic-slug>/
      _index.md                  # Summary with cross-references
      01-<angle-slug>.md
      02-<angle-slug>.md
      ...
```

### Agent Contract

Each subagent receives:
- A focused sub-question (one research angle)
- Instructions to use `WebFetch` and `WebSearch`
- A target output file path (absolute)
- Format: structured markdown with sections for key findings, details, sources, and raw data

### Index Generation

After all agents complete, a final sequential pass:
1. Reads all output files in the topic directory
2. Synthesizes an `_index.md` with: executive summary, key findings across all angles, cross-references between files, and suggested follow-up questions

## Constraints

- No external dependencies (no npm, no Python, no APIs beyond what Claude Code provides)
- All output is markdown
- Agents are `general-purpose` subagents only
- Target 15-25 agents per run (can be fewer for narrow topics)
- File names are numbered and slugified for easy reference
