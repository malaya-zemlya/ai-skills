# ai-skills

Agent skills and plugins for [Claude Code](https://claude.com/claude-code), published as a plugin marketplace.

## Install

Add the marketplace once:

```
/plugin marketplace add malaya-zemlya/ai-skills
```

Then install whichever plugins you want:

```
/plugin install human-plan
```

## Plugins

### human-plan

Implementation plans a human can meaningfully review and approve, instead of goal-oriented task lists.

A plan is ready for a human when a reviewer can reconstruct the intended structure of the code from the prose alone, and see every decision and assumption laid out as something they can accept or flip. The quality bar for every implementation section is the **two-developers test**: if two developers independently implemented this section, would their code be structurally the same?

| Command | What it does |
| --- | --- |
| `/human-plan <what you want built>` | Requirements elicitation one question at a time, then a full plan, an adversarial critic pass, and a walkthrough. |
| `/human-plan-critique <path/to/plan.md>` | Runs the critic alone over an existing plan document, including one you did not write with Claude. |

The `plan-for-humans` skill also triggers on its own in planning mode or when you ask for a plan or design doc. Full details in [`human-plan/README.md`](human-plan/README.md).

## Repository layout

Each plugin is a self-contained directory listed in `.claude-plugin/marketplace.json`. To add another, create its directory with a `.claude-plugin/plugin.json` and append an entry to the marketplace manifest.

```
.claude-plugin/marketplace.json   the marketplace index
human-plan/                       one plugin
```
