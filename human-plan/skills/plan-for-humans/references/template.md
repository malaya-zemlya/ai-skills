# Plan template and document structure

The structure of a plan document written under the `plan-for-humans` skill. Every implementation section is held to the two-developers test: if two developers independently implemented this section, would their code be structurally the same?

## Plan template

### 1. Goal and scope
One short paragraph: what is being built or changed, and for whom. Then an explicit **non-goals** list, so the boundary of the work is visible. Boundaries the user set in phase 0 are stated as settled; any remaining boundary you chose goes to Open Questions.

### 2. Architecture (prose component map)
Describe the system in prose, at the level a UML diagram would convey. For each component give:
- **Name and responsibility** — one sentence.
- **State it owns** — the actual data structures: shapes, types, key fields.
- **Public interface** — the operations it exposes, with what each validates, mutates, and returns.
- **Who calls it, and what it is allowed to know about** — make the dependency direction explicit.

Then state globally: where state lives, which single paths are allowed to mutate it, and how changes propagate to the system's outputs (screen, API response, file, message — whatever the domain produces).

### 3. Per-component implementation
For each component, one level deeper: concrete mechanisms, wiring, error and edge handling. Apply the two-developers test to every subsection.

Clarity devices for complex logic — pick whichever fits, in plain text (prose, tables, and pseudocode render everywhere; graphical diagram syntaxes depend on the harness, so express structure in these text forms):
- **Non-trivial algorithms:** high-level pseudocode, numbered steps.
- **Non-trivial state machines:** a table — rows are states, columns are events, cells are action → next state.
- Also useful: worked examples tracing one input through the system; invariants stated as "always true" sentences; before/after data snapshots for tricky transformations.

### 3a. Change plans: deltas and the reviewer's mental model

When the plan modifies an existing system (most plans do), write sections 2–3 as deltas against the current code, and add two things:

- **Verdict labels in the component map.** Every existing component touched by or adjacent to the change gets one: *unchanged / extended / modified / replaced / removed*. "Unchanged" is information too — it tells the reviewer which parts of their knowledge stay valid.
- **Change manifest** (part of section 3): the concrete edits, at the granularity a diff reader would recognize. "Add field `level` to session state." "Rename `check()` to `evaluate()`." "Move win detection from controller into `Game`." "Thread the new `rng` argument through `chooseMove` → `chooseBest` → `best`" — a threaded parameter's route is named function by function. Include the refactorings done *in service of* the change, since those touch code the reviewer thought was settled.
- **Mental-model sync list**: every statement about the current system that stops being true after this change, stated as before → after. "`render` takes one argument → takes `(game, session)`." "Settings controls are always active → disabled mid-round." The reviewer walks in with a model of today's code; this list is the complete set of updates their model needs. Behavior changes to existing features belong here even when they look like implementation detail — anything a person who knew the old code could be surprised by.

### 4. Acceptance criteria
For each feature: a short definition of done — observable conditions a reviewer can check ("entering a mark in a full cell leaves the board unchanged and keeps the turn"), including the relevant error and edge behaviors. These become the checklist for accepting the implementation, and the seed for the tests.

### 5. Testing plan
The implementation is designed for testability from the start: pure logic in components with narrow interfaces, effects at the edges — that structure is what keeps tests short.
- Map acceptance criteria to concrete tests; state the kind (unit, integration, end-to-end) and what each asserts.
- For tricky cases, sketch the test implementation: setup, the shape of the fixture or harness, the assertion.
- A test complex enough to need its own scaffolding or verification is a design signal: prefer reshaping the component so the test becomes simple, and record that fork in the decision register.

### 6. Decision register
The genuine forks in the design. Each entry: **decision** (stated plainly), **alternatives considered** (every genuinely viable one — a real three-way fork gets three entries in the list, keeping the full option space visible for the walkthrough), **why the chosen option** — a real rationale, plus what changes if the reviewer flips it. The decision register is the home for all alternatives and roads considered; the rest of the plan simply states the chosen design.

### 7. Open questions
Whatever survived phase 0: forks the user deferred, features you propose, tradeoffs with no technical winner. Each phrased as a direct question with the consequence of each answer in one line. During the walkthrough these are asked one at a time, same protocol as phase 0.

## Scaling up: deep modules, split documents

When the plan outgrows one sitting's read (a few pages), split it along module boundaries — the document structure mirrors, and tests, the encapsulation of the design:

- **Prefer deep modules**: few components, each with a small, fully specified interface hiding substantial functionality. This is both a design preference (less coupling to review, fewer decisions leaking across boundaries) and what makes the split possible at all.
- **Main plan document**: goal/scope, the component map with every interface specified to the two-developers standard *at the boundary* (a module could be implemented from its interface spec plus its own doc, touching nothing else), system-level acceptance criteria, the decision register, open questions. This document alone is the approval surface.
- **One module doc per deep component**, referenced from the main plan: the module's internal design at full section-3 depth, its internal tests, and module-local decisions. Two independent tests promote a decision to the main register: **structural** — it is visible at an interface or its flip touches a second module — and **human salience** — its consequences reach outside the code, however encapsulated the mechanism is. Where data is stored and how long it lives (cookie vs. localStorage vs. server), external services and dependencies taken on, security- and privacy-relevant mechanisms, anything bearing cost: these are user decisions even when they are implementation details by every structural measure, and when requirements leave them open they are *asked*, under the same question protocol as phase 0.
- **Encapsulation organizes reading, never access.** Module docs are written for the same human reader, under the same style rules, to the same two-developers standard — the full design is understandable end to end by a reader who chooses to open everything. The split economizes the reviewer's attention; it hides nothing from them.
- **The extraction is a design test**: a module whose internals cannot leave the main doc without making it ambiguous has a leaky interface — fix the design, then split.
- **Physical form follows the harness.** Where files can be written: a `plan/` directory, `plan.md` plus `modules/<name>.md`. Where plan mode is text-only: one document with the module docs as appendices and internal links — the same structure, inlined. Where writing files requires leaving plan mode: say so in one line and ask at the moment the threshold is crossed.
- **Walkthrough scope**: the walkthrough covers the main document. Module docs are reviewed on demand — progressive disclosure is the point, and "reviewed the contracts, opened two of five module docs" is an honest, meaningful approval.

