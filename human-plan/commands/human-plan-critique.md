---
description: Run the plan-for-humans critic over an existing plan document
model: opus
---

Review an existing plan document — one you did not necessarily write — against the `plan-for-humans` quality bar.

1. Read the plan named below. If `$ARGUMENTS` is empty, ask which document to review.
2. Locate `references/critic.md` in the `plan-for-humans` skill directory and note its absolute path.
3. Dispatch the `plan-critic` agent with four inputs: the user's original request (ask if you do not have it, or say it is unavailable), whatever elicitation record exists, the plan's path, and the checklist path.
4. Present the findings verbatim, grouped by severity, followed by the agent's verdict.

Report the findings. Do not revise the plan unless the user asks.

Plan to review: $ARGUMENTS
