# human-plan

A Claude Code plugin for writing implementation plans that a human can meaningfully review and approve, instead of goal-oriented task lists.

A plan is ready for a human when a reviewer can (a) reconstruct the intended structure of the code from the prose alone, and (b) see every decision and assumption laid out as something they can accept or flip.

The quality bar for every implementation section:

> **The two-developers test:** if two developers independently implemented this section, would their code be structurally the same?

## Install

```
/plugin marketplace add malaya-zemlya/ai-skills
/plugin install human-plan
```

## Use

| Command | What it does |
| --- | --- |
| `/human-plan <what you want built>` | Full run: requirements elicitation, draft, critic pass, walkthrough. |
| `/human-plan-critique <path/to/plan.md>` | Runs the critic alone over a plan document — including one you did not write with Claude. |

The `plan-for-humans` skill also triggers on its own whenever you enter planning mode, ask for a plan or design doc, or ask "how would you implement this". The slash commands are for invoking it deliberately.

## How a run goes

1. **Phase 0 — elicitation.** The requirement space is walked as a decision tree, one question at a time, with the tradeoffs of each option printed before the question. Requirements belong to you; implementation forks are decided for you and recorded.
2. **Draft.** Seven sections: goal and scope, architecture as a prose component map, per-component implementation, acceptance criteria, testing plan, decision register, open questions. Plans that modify existing code additionally carry verdict labels, a change manifest, and a mental-model sync list.
3. **Critic pass.** The `plan-critic` subagent reviews the draft against a nine-category checklist of failure modes — abstraction drift, scope invention, silent assumptions, testability gaps, and so on. The checklist is deliberately kept out of the drafting context; the separation is what makes the critique work. Silent by default.
4. **Walkthrough.** You are walked through the decision register and open questions entry by entry, and can flip any of them or ask for more depth anywhere.
5. **Consolidation, then stop.** Every amendment is folded into one clean document. Implementation begins only when you say so.

## Layout

```
.claude-plugin/plugin.json          plugin manifest
commands/human-plan.md              /human-plan
commands/human-plan-critique.md     /human-plan-critique
agents/plan-critic.md               the critic, as a first-class subagent
skills/plan-for-humans/
  SKILL.md                          purpose, phase 0, style, critic dispatch, walkthrough
  references/template.md            the seven-section template + split-document rules
  references/critic.md              the nine-category critic checklist
```

`SKILL.md` holds the protocol and stays small so the skill is cheap to trigger; the template and the checklist load only when drafting and critiquing actually start.
