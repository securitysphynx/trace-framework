# TRACE Skills for AI Assistants

Practical prompts and skills for applying the TRACE framework with AI assistants like Claude Code, Cursor, and other LLM-powered development tools.

## What These Are

TRACE skills are system prompts and structured workflows that help security practitioners apply the TRACE framework to real evaluation and investigation scenarios. Instead of reading the framework and trying to apply it manually, these skills guide you through the process interactively.

## Available Skills

| Skill | Purpose | Use When |
|-------|---------|----------|
| [trace-evaluate](trace-evaluate.md) | Structured AI system evaluation | Evaluating a new AI tool, vendor product, or deployment |
| [trace-investigate](trace-investigate.md) | Incident analysis through TRACE lens | An AI system failed, missed something, or behaved unexpectedly |
| [trace-vendor](trace-vendor.md) | Procurement and vendor assessment | Writing RFP requirements or evaluating vendor claims |

## How to Use

### Claude Code

Copy the skill content into your project's `CLAUDE.md` or import as a skill:

```bash
# Add to your project
cp trace-evaluate.md /path/to/your/project/.claude/skills/
```

Then invoke with `/trace-evaluate` or reference in conversation.

### Cursor / Other AI IDEs

Add the skill content to your system prompt or rules file. The skills are written as markdown that any LLM can follow.

### Standalone Use

Paste the skill content at the start of a conversation with any capable LLM, then provide your evaluation context.

## Design Principles

These skills are designed to:

1. **Ask, not tell** — Guide users through questions rather than providing answers
2. **Surface gaps** — Reveal what you don't know about an AI system
3. **Produce artifacts** — Generate documentation you can share with stakeholders
4. **Stay lightweight** — No dependencies, just markdown prompts

## Contributing

These skills evolve with practitioner feedback. If you apply TRACE in your work and find gaps, open an issue or PR.

---

*Part of the [TRACE Framework](../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
