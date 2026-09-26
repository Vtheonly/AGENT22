# AGENT22

## GLM / Z.ai Agent Mode — Multi-Agent Coding Playground

**AGENT22 is a controlled playground for GLM running in Z.ai Agent Mode.**

The purpose of this repository is to make GLM operate as an active software engineer rather than a text-only coding assistant.

When GLM is given a real coding project inside this repository, it is expected to **use its available agent/tooling infrastructure, invoke the external coding agents, execute the work, verify the result, and measure what happened**.

The central question is:

> **For a given software-engineering job, which agent workflow lets GLM complete the work fastest, with the fewest unnecessary actions, while still producing a correct and high-quality result?**

This is not a repository for one fixed application. It is a **playground, benchmark harness, and experiment log** for GLM's use of coding agents.

---

# 1. The Core Idea

Do not think of the system as:

```text
User -> GLM -> code
```

Think of it as:

```text
                         USER TASK
                             |
                             v
                    GLM / Z.ai Agent Mode
                             |
                  understand the real task
                             |
                    inspect the repository
                             |
                 choose/dispatch agent tools
                             |
        +--------------------+--------------------+
        |                    |                    |
        v                    v                    v
       Pi                OpenCode             Hermes
        |                    |                    |
        +--------------------+--------------------+
                             |
                     Ponytail when useful
                  (behavior / policy modifier)
                             |
                             v
                     real file + shell work
                             |
                             v
                     tests / verification
                             |
                             v
                     measured result
```

The LLM is the reasoning engine.

The agents are execution/orchestration environments.

Ponytail is primarily a behavior modifier that can be attached to supported hosts; it is **not** counted as an independent coding harness in the same sense as Pi, OpenCode, or Hermes.

GLM must therefore distinguish between:

1. **Model capability** — what GLM can reason about and generate.
2. **Agent harness capability** — how Pi, OpenCode, or Hermes handle tools, state, planning, memory, delegation, and execution.
3. **Agent policy/plugin** — for example Ponytail's minimal-code/anti-overengineering rules.
4. **Execution environment** — filesystem, shell, Git, network, containers, permissions, and other available tools.

Do not confuse these layers when interpreting an experiment.

---

# 2. This Playground Is Specifically for GLM

AGENT22 is **not** a generic benchmark specification that happens to mention GLM.

It is specifically designed around the following operating model:

```text
GLM
  |
  +-- Z.ai Agent Mode
  |
  +-- tool calls
  |
  +-- external coding agents
  |
  +-- real repository changes
  |
  +-- verification
  |
  +-- measured comparison
```

GLM should behave like an engineer that can decide:

- when to inspect the code itself;
- when to use a coding agent;
- which coding agent is most appropriate;
- when to use Ponytail as a constraint against overengineering;
- when to stop investigating and implement;
- when to test;
- when to retry after failure;
- when to delegate;
- when not to waste time on unnecessary tool calls.

The goal is **fast and correct**, not merely fast and not merely verbose.

---

# 3. The Agents Under Test

## Pi

Treat **Pi** as the lightweight, surgical coding harness.

Prefer Pi for:

- one-file fixes;
- localized bug fixes;
- small edits;
- simple refactors;
- direct shell-driven tasks;
- quick edit/test cycles;
- small scripts;
- headless coding workflows;
- tasks where the required change is already well understood.

The experiment should ask whether Pi can finish these jobs with a very small tool loop instead of introducing unnecessary planning or abstraction.

### Typical Pi task

```text
Find the broken null check in src/auth.ts, fix it using the existing helper,
run the relevant tests, and stop when the tests pass.
```

---

## OpenCode

Treat **OpenCode** as the main general-purpose repository engineering harness.

Prefer OpenCode for:

- unfamiliar repositories;
- multi-file features;
- repository-wide changes;
- architectural changes;
- medium/large refactors;
- frontend + backend work;
- database/schema changes;
- framework migrations;
- dependency migrations;
- LSP-heavy tasks;
- tasks where planning and implementation should be separated.

The experiment should observe whether OpenCode's stronger repository-level orchestration reduces mistakes on larger jobs without creating unnecessary overhead on tiny jobs.

### Typical OpenCode task

```text
Understand the existing authentication flow, add role-based access control,
update the relevant database/schema code, update the API and UI, add tests,
and verify the complete build.
```

---

## Hermes Agent

Treat **Hermes** as the autonomous and long-horizon engineering harness.

Prefer Hermes for:

- difficult debugging;
- root-cause analysis;
- long multi-step work;
- autonomous repository maintenance;
- delegation to subagents;
- parallel investigation;
- persistent memory workflows;
- GitHub/PR workflows;
- recurring maintenance;
- tasks that benefit from a persistent worker rather than one short edit loop.

The experiment should measure whether Hermes reduces **human intervention** and solves difficult workflows more reliably, not simply whether it makes more tool calls.

### Typical Hermes task

```text
Investigate why the production synchronization job intermittently duplicates
records. Reproduce the failure, trace the data flow, identify the root cause,
fix it, add regression coverage, run the relevant test suite, and document the
cause and fix.
```

---

## Ponytail

Treat **Ponytail as a behavior modifier**, not as a fourth independent coding harness.

Its purpose is to push the host toward the smallest implementation that still fully satisfies the task.

The benchmark should therefore test conditions such as:

```text
Pi
Pi + Ponytail

OpenCode
OpenCode + Ponytail

Hermes
Hermes + Ponytail
```

Use Ponytail when the experiment is specifically testing:

- overengineering;
- unnecessary abstractions;
- dependency sprawl;
- redundant wrappers;
- unnecessary code generation;
- diff size and scope discipline;
- codebase simplification.

Do **not** interpret “smaller diff” as automatically better.

A smaller implementation is only an improvement when it preserves:

- correctness;
- security;
- validation;
- error handling;
- accessibility;
- tests;
- required behavior;
- maintainability appropriate to the task.

---

# 4. Mandatory GLM Agent Protocol

This section is the most important part of the repository.

The following behavior is mandatory for GLM when it is operating inside AGENT22.

## Rule 1 — Do the work; do not just explain the work

When the user gives a coding task, GLM must operate the environment and the available tools instead of responding with a proposed patch only.

If the task requires code changes, the expected sequence is normally:

```text
inspect -> reason -> act -> test -> inspect result -> fix if necessary -> verify -> report
```

A textual explanation is not considered task completion if the repository was supposed to be changed.

---

## Rule 2 — Use the real agent tools

When the task is designated as an AGENT22 benchmark/build task, GLM must actually invoke the available external agent tooling.

Do not simulate Pi, OpenCode, Hermes, or Ponytail by merely writing a prompt that pretends another agent ran.

A run counts only when the real tool/agent was invoked and its output was observed.

---

## Rule 3 — Use all requested agents for comparison tasks

When the user says that the task must test **all agents**, GLM must run all available requested configurations.

The default comparison matrix is:

```text
1. Pi
2. OpenCode
3. Hermes
4. Pi + Ponytail
5. OpenCode + Ponytail
6. Hermes + Ponytail
```

If a configuration is unavailable, do not silently replace it.

Record:

```text
STATUS: NOT RUN
REASON: <actual reason>
```

If the user asks only for a single production implementation rather than a benchmark, GLM may use the agent it determines is appropriate; however, if the user explicitly says “use all agents,” the full matrix is mandatory.

---

## Rule 4 — The same job must be comparable

For fair comparisons, every agent should receive:

- the same task;
- the same acceptance criteria;
- the same baseline repository state;
- equivalent environment access;
- equivalent required information;
- equivalent test requirements.

Use isolated worktrees, branches, containers, or clean repository copies whenever possible.

Do not let Pi's changes become OpenCode's starting point, for example.

---

## Rule 5 — Never optimize for tool-call count alone

The objective is **efficient work**.

Efficient means:

```text
few unnecessary actions
+
fast useful actions
+
correct implementation
+
strong verification
```

Do not make fake or pointless tool calls merely to prove that tools were used.

At the same time, do not avoid necessary tools just to appear fast.

A 20-tool investigation that prevents a production regression can be better than a 4-tool guess that breaks the system.

---

## Rule 6 — Use tools according to the job

### Tiny task

Prefer:

```text
Pi
```

Example:

```text
rename one function -> update references -> run targeted test
```

### Small task with overengineering risk

Prefer:

```text
Pi + Ponytail
```

### Medium/large repository feature

Prefer:

```text
OpenCode
```

### Large feature where complexity control matters

Prefer:

```text
OpenCode + Ponytail
```

### Difficult root-cause debugging

Prefer:

```text
Hermes
```

### Autonomous/long-running workflow

Prefer:

```text
Hermes
```

### Autonomous work where unnecessary complexity is a concern

Prefer:

```text
Hermes + Ponytail
```

These are **default hypotheses**, not benchmark conclusions. Actual results must be measured.

---

# 5. Mandatory Tool Usage Strategy

GLM must use the available Z.ai Agent Mode tools and external agent tooling whenever they are relevant to the job.

The agent should generally use the following progression.

## Phase A — Understand

First determine:

- what the user actually asked for;
- what files/modules are likely involved;
- what constraints already exist;
- whether there are repository instructions;
- what tests/build commands exist;
- whether the task is local, repository-wide, or autonomous.

Do not blindly edit the first file that looks relevant.

## Phase B — Choose execution strategy

Choose between:

```text
Direct GLM tool use
Pi
OpenCode
Hermes
Ponytail-modified host
```

based on task characteristics.

For explicit benchmark tasks, do not skip the required comparison matrix.

## Phase C — Execute

Use the selected agent's real tools to:

- inspect files;
- search the repository;
- edit code;
- execute shell commands;
- run tests;
- use Git;
- use GitHub where required;
- use browser/web tools when genuinely necessary;
- use subagents where the host supports them and the task benefits from them.

## Phase D — Verify

Do not stop at “the code looks right.”

Verify through the strongest practical checks available:

```text
test suite
lint
typecheck
build
runtime checks
acceptance tests
```

Use task-appropriate checks rather than running everything blindly when a targeted verification is sufficient.

## Phase E — Measure

For benchmark runs, collect as much objective data as the environment exposes.

Preferred metrics include:

- wall-clock duration;
- input tokens;
- output tokens;
- reasoning tokens when exposed;
- total tokens;
- tool calls;
- shell commands;
- file reads;
- file edits;
- failed tool calls;
- retries;
- test runs;
- human interventions;
- files changed;
- lines added/deleted;
- final diff size;
- dependencies added;
- regressions;
- final test status.

## Phase F — Record

Write the result to a machine-readable benchmark record where the project provides one.

Never claim a metric that was not actually observed.

Use `NOT MEASURED` when the infrastructure does not expose a value.

---

# 6. What “Fast” Means in AGENT22

Fast does **not** mean:

```text
minimum thinking
minimum tool calls
minimum code
```

Fast means:

```text
minimum wasted effort required to reach a verified correct result
```

The benchmark should therefore distinguish:

### Useful latency

Time spent on actions that materially help solve the task.

### Wasted latency

Time spent on:

- redundant reading;
- repeated failed commands that could have been avoided;
- unnecessary planning;
- unnecessary abstraction;
- unnecessary agent delegation;
- irrelevant investigation;
- excessive tool chatter;
- rebuilding information already available.

The desired agent is not the one that simply acts the fastest. It is the one that reaches a **verified correct result with the least wasted work**.

---

# 7. What “Right” Means

Correctness comes before raw speed.

A successful implementation should satisfy:

```text
requirements
+
existing architecture
+
existing conventions
+
required tests
+
security/validation requirements
+
no unacceptable regressions
```

A fast incorrect patch is a failed run.

A small but incomplete implementation is also a failed run.

---

# 8. Task-Type Decision Matrix

| Job | First workflow to test | Why |
|---|---|---|
| One-line/local fix | Pi | Minimal execution overhead |
| Small bug | Pi | Fast inspect/edit/test loop |
| Small bug with overengineering risk | Pi + Ponytail | Minimal implementation pressure |
| Simple utility/script | Pi | Direct shell/editor workflow |
| Small refactor | Pi | Surgical repository change |
| Unfamiliar repository | OpenCode | Better exploration/planning fit |
| Medium feature | OpenCode | General repository engineering |
| Large multi-file feature | OpenCode | Broader context and structured execution |
| Architecture change | OpenCode | Planning + implementation |
| Framework migration | OpenCode | Cross-cutting change management |
| Database/schema migration | OpenCode | Multi-layer consistency |
| Full-stack feature | OpenCode | Frontend/backend breadth |
| Large change + overengineering concern | OpenCode + Ponytail | Breadth plus scope discipline |
| Hard debugging | Hermes | Long investigative loop |
| Root-cause analysis | Hermes | Investigation before implementation |
| Multi-step autonomous job | Hermes | Long-horizon execution |
| Delegated investigation | Hermes | Subagent-oriented workflow |
| Recurring maintenance | Hermes | Persistent/autonomous operation |
| GitHub/PR automation | Hermes | Workflow automation |
| Overengineering review | Ponytail on the relevant host | Specialized complexity reduction |
| Whole-repo simplification | Ponytail + repository-capable host | Finds unnecessary complexity without treating smaller code as the only metric |

Again: **this table is the initial routing hypothesis. Benchmark data may overturn it.**

---

# 9. The Benchmark Must Be Hard Enough to Matter

Do not judge the agents using only trivial toy problems.

Useful AGENT22 tasks include:

```text
01. one-line fix
02. localized bug
03. medium bug
04. difficult root-cause bug
05. small feature
06. medium feature
07. large feature
08. cross-module feature
09. refactor
10. architectural refactor
11. dependency migration
12. framework migration
13. database migration
14. API integration
15. authentication/authorization change
16. security fix
17. performance investigation
18. performance optimization
19. flaky-test repair
20. unit/integration/E2E test generation
21. CI failure diagnosis
22. code review
23. security audit
24. overengineering audit
25. documentation update
26. unfamiliar-codebase exploration
27. multi-repository change
28. multi-agent task
29. long autonomous task
30. recurring maintenance workflow
```

The same repository should contain tasks of different sizes because an agent that is excellent at a 5-minute change may not be excellent at a 3-hour investigation.

---

# 10. Required Experiment Record

Every benchmark run should be identifiable by a unique run ID.

A recommended record shape is:

```yaml
TASK_ID: task-001
RUN_ID: run-2026-09-26-001
AGENT: opencode
PLUGIN: ponytail
MODEL: <exact GLM model>
MODE: Z.ai Agent Mode
STATUS: SUCCESS
BASE_COMMIT: <sha>
DURATION_SECONDS: <value or NOT MEASURED>
INPUT_TOKENS: <value or NOT MEASURED>
OUTPUT_TOKENS: <value or NOT MEASURED>
REASONING_TOKENS: <value or NOT MEASURED>
TOTAL_TOKENS: <value or NOT MEASURED>
TOOL_CALLS: <value or NOT MEASURED>
FAILED_TOOL_CALLS: <value or NOT MEASURED>
RETRIES: <value or NOT MEASURED>
FILES_CHANGED: <value or NOT MEASURED>
LINES_ADDED: <value or NOT MEASURED>
LINES_DELETED: <value or NOT MEASURED>
DEPENDENCIES_ADDED: <value or NOT MEASURED>
HUMAN_INTERVENTIONS: <value or NOT MEASURED>
TEST_STATUS: PASS
BUILD_STATUS: PASS
REGRESSIONS: 0
NOTES: <observed facts>
```

---

# 11. Never Fake Results

This is a strict experiment.

Never invent:

- tokens;
- timings;
- agent calls;
- tool calls;
- test results;
- benchmark scores;
- success rates;
- cost estimates;
- model configurations;
- claims that another agent performed an action.

If something was not run:

```text
NOT RUN
```

If something could not be measured:

```text
NOT MEASURED
```

This rule exists so that the playground remains useful for real comparison instead of becoming a collection of opinions.

---

# 12. The Future Workflow

The intended future interaction is simple.

The user gives GLM a real development task.

For example:

```text
Build feature X in this repository.
Use all available AGENT22 agents to test which workflow completes it best.
```

GLM should then:

```text
1. inspect AGENT22 instructions;
2. inspect the repository/task;
3. understand the acceptance criteria;
4. dispatch Pi;
5. dispatch OpenCode;
6. dispatch Hermes;
7. dispatch Ponytail-modified variants when required;
8. keep the runs isolated;
9. let each agent perform real work;
10. run equivalent verification;
11. collect objective metrics;
12. compare the observations;
13. report which workflow was most suitable for this specific job;
14. preserve the raw results.
```

The goal is not to permanently crown one agent.

The goal is to build evidence for **task-specific agent routing**.

---

# 13. The Core Principle

> **GLM should not merely answer the coding problem. GLM should use the available agent systems as execution tools, perform the real work, verify it, measure it, and learn which workflow is most efficient for each type of engineering job.**

The desired behavior is:

```text
UNDERSTAND
   -> CHOOSE
   -> USE THE REAL AGENTS
   -> EXECUTE
   -> VERIFY
   -> MEASURE
   -> COMPARE
   -> IMPROVE ROUTING
```

That is what AGENT22 is for.
