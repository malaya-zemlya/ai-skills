---
name: plan-for-humans
description: Write implementation plans that a human can meaningfully review and approve, instead of goal-oriented task lists. Use this skill whenever entering planning mode, writing a plan, design doc, or proposal for a code change, or whenever the user asks for a plan, asks "how would you implement this", or is about to approve AI-generated work. Applies to any codebase or domain (backend, frontend, infra, data, CLI).
---

# Plans for Humans

## Purpose

A plan is ready for a human when a reviewer can (a) reconstruct the intended structure of the code from the prose alone, and (b) see every decision and assumption laid out as something they can accept or flip. This skill extracts the real requirements from the user's head, produces that kind of plan, and walks the reviewer through it.

The quality bar for every implementation section:

> **The two-developers test:** if two developers independently implemented this section, would their code be structurally the same? A section passes when the answer is yes; until then, keep adding the decisions that would make it yes.

## Phase 0: requirements elicitation

Before drafting, resolve the requirements. The goal is to get the user's actual spec out of their head — the plan describes what they want, and they are the only source of that.

- Walk the requirement space as a decision tree: each answer typically opens follow-up branches (a "computer opponent: yes" answer opens difficulty, who moves first, and so on). Keep going until every requirement-level fork is settled or explicitly deferred by the user.
- Ask about anything unclear, and ask liberally — at the requirements level, over-asking is the correct failure direction.
- Every question carries enough context for a meaningful decision: what the choice affects, the realistic options, and each option's pros and cons in a line or two. Print all of that context as regular output *before* the tool call; the question tool itself carries only a short question sentence and short option labels, because small screens truncate long question text.
- **One question at a time.** Ask, wait for the answer, let it reshape the tree, then ask the next. The user needs room to ponder each fork.
- Use the harness's structured question tool (e.g. `AskUserQuestion`) when one exists; fall back to plain conversational questions otherwise. Offered options are suggestions: invite free-form answers, and expect some of the best answers to be off-menu — they may reshape the tree.

**Implementation-level questions follow a different budget.** Implementing to spec is your job, so decide most implementation forks yourself and record them in the decision register. Bring one to the user only when options genuinely diverge in consequences they'd care about (operational cost, extensibility, dependencies) and the requirements leave the choice open. When you do ask, give the same context — options, tradeoffs — plus your recommendation and why. When you're confident one option is best, choose it and record it.

## Plan template

The seven-section template — goal and scope, architecture, per-component implementation, change deltas, acceptance criteria, testing plan, decision register, open questions — and the rules for splitting a plan across documents when it outgrows one sitting's read, are in `references/template.md`. Read that file before drafting.

## Writing style

- **State each decision as a plain assertion.** "State is a 9-element array." The sentence is complete once the choice is named; rationale and alternatives live in the decision register.
- **Name the mechanism behind every behavior.** For each thing the system does, say which component does it, with what data, triggered by what.
- **Keep the requested scope in sections 1–5, and your proposals in section 7.**
- **Self-contained prose.** The reader sees only the plan document — reasoning traces and your side of prior context are invisible to them. Define every term of art on first use, use established vocabulary, and expand abbreviations once. If a term you coined is doing real work, define it where it first appears.
- **Stay domain-general.** Screens, endpoints, tables, CLIs are all "the system's inputs and outputs"; the same template applies.

## Critic pass

Draft the full plan first. Then run a critic over it as a separate step. Generation and critique happen in separate steps — the split is the mechanism that makes the critique effective.

- Dispatch the `plan-critic` agent with exactly four inputs: the user's request, the decisions agreed during elicitation, the draft plan, and the absolute path to this skill's `references/critic.md`. The agent reads that checklist and returns findings. Silent by default; show the critique if the user asks for debug mode.
- Where that agent is unavailable, spawn a generic subagent with the same four inputs. Where no subagents exist at all, finish the draft completely, then read `references/critic.md` yourself and do a dedicated review pass.

Revise on the critic's findings, then present.

## Walkthrough, consolidation, stop

After presenting the plan:

1. Walk the user through the **Decision register** and **Open Questions**, entry by entry — one item at a time, via the question protocol above — inviting objections. These two sections are the review surface.
2. Edit the plan as decisions are made or flipped. The user can also ask for more detail on any part — a section, a module, a single mechanism. Deepen that spot in place (in the main doc or the module doc it lives in) to whatever depth they ask for; fleshing out on request is part of review, and the added text is held to the same standards as the rest.
3. When the walkthrough ends, fold every amendment into one clean, complete document and deliver it. The reader approves a document, and only a consolidated one can be read whole — a plan scattered across chat messages and diffs is unreviewable.
4. Then stop. The user reads and approves on their own schedule; approval means "I read this document and reviewed its N decisions". Delivering the document is the end of the planning task — implementation begins only when the user explicitly starts it.

## Calibration example (abridged)

Request: "a tic-tac-toe game in the browser." Phase 0 opens with (one at a time): "Two humans at one screen, or a computer opponent? Opponent adds an AI module and a difficulty question; two-human is smaller." A reviewable architecture section then reads:

> `Game` owns `board` (9-element array, `null|'X'|'O'`), `currentPlayer`, and `status` (`'playing'|'won'|'draw'`). Its single mutation entry point is `play(index)`: validates the cell is empty and status is `'playing'`, places the mark, evaluates the 8 win lines internally, and returns whether the move was accepted. `BoardView` builds 9 buttons index-aligned with the board and exposes `render(game)`, a full idempotent sync from state. A thin controller wires clicks (one delegated listener reading `data-index`) to `game.play`, then `render`. Dependency direction: view reads game state; game knows nothing of the view.

Acceptance criterion: "Clicking a filled cell leaves board, turn, and status unchanged." Test: "unit — `play(4)` twice returns `false` the second time and `board[4]` still holds the first mark; runs against `Game` alone, without a browser." Decision register entry: "Full re-render on each move. Alternative: patch only the changed cell. Chosen because the board is 9 cells, so a full sync is trivially cheap and makes the view stateless; flipping this adds dirty-tracking to BoardView."
