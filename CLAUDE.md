# CLAUDE.md — work protocol (mandatory, no exceptions)

Applies to every session and every task in this repository. Nothing waives it.

Every piece of work, from a release to a one-line edit, is decomposed into the hierarchy below and
driven through the four-beat cycle. Small tasks get a *short* cycle, not *no* cycle. "It's obvious"
and "it's a one-liner" are not exemptions; they are the cases where skipping the protocol costs the
most, because implementing before diagnosing is how the expensive mistakes get made.

## 1. Decomposition hierarchy

| Level | Span | Owns |
|---|---|---|
| **Stage** | A capability that changes what the system can do | A pre-registered contract: primary metric, hard constraints, minimum useful improvement, kill criteria |
| **Phase** | A coherent slice of a stage, independently verifiable | One measurable exit condition |
| **Task** | One change with one reason to change | A hypothesis and the check that falsifies it |
| **Atomic action** | One indivisible step: one edit, one command, one measurement | Nothing — it either happened or it did not |

Rules:

- **No level is skipped.** A stage is split into phases before it is worked on; a phase is split into
  tasks before it is "just done".
- **An atomic action is reversible and inspectable in isolation.** If a step cannot be checked on its
  own, it is a task and must be split further.
- **One task, one reason.** Bug fix + refactor + feature is three tasks. Split before starting.
- **Declare the decomposition before executing it.** State the stage/phase/task at the start, so a
  wrong level is caught before effort is spent at it.
- **Width before depth.** Where a phase admits several directions, enumerate them (refine the
  incumbent / change approach / simplify / question an assumption), say which one this task takes,
  and why the others are deferred. Do not silently pick one.

## 2. The cycle: root cause analysis → plan → implement → verify

Run at every level. The cycle is **ordered and gated**: a beat does not start until the previous beat
has produced its artifact. Never jump to implementation.

### Beat 1 — Root cause analysis

- State the symptom as evidence: a number, an error, a diff, a log line. Not "it's slow" — the
  measured figure and where it came from.
- Reproduce it. An unreproduced symptom is not a root cause; say so explicitly.
- Ask why until the answer is a mechanism, not a restatement. Stop at the layer you can change.
- Name the alternative explanations considered and the evidence ruling each out. Anything not ruled
  out stays open and the plan must account for it.
- Read the primary source — the real error text, the actual data file, the library's own code —
  before reasoning from memory about what it probably says. Diagnosing from a prior belief instead of
  the message in front of you is the most expensive failure mode there is.

**Artifact:** a one-paragraph causal statement — *symptom → mechanism → evidence → what remains
uncertain*. No plan is written without it.

### Beat 2 — Plan

- Decompose into the hierarchy above, down to atomic actions for the part being done now.
- **Pre-register the bar before running anything**: primary metric, its direction, hard constraints,
  the minimum improvement that counts as useful, and the kill criteria that end the attempt. A bar
  chosen after seeing the result is not a bar.
- Fix the budget — time, compute, attempts — and say what happens when it runs out.
- Say what the plan does *not* cover, and name the assumptions it rests on.
- Prefer the smallest plan that can be falsified. If a smaller version would settle the question, do
  that first.

**Artifact:** the written plan, with the pre-registered bar and budget, recorded *before* the work.

### Beat 3 — Implement

- Execute only what the plan contains. A discovery that demands more scope ends this beat and
  re-enters Beat 1 with the new evidence; it is not absorbed silently.
- Keep each atomic action small enough to inspect and undo.
- Match the surrounding code: its idiom, naming, and comment density.
- Record failures as they happen. A deleted failed attempt costs the same and teaches nothing; a
  recorded one stops it from being retried.
- When the plan turns out to be wrong, stop and say so. Do not patch toward making a broken plan
  appear to work.

**Artifact:** the change, plus the running record of what was tried, including what failed.

### Beat 4 — Verify

- Check against the bar from Beat 2, as written. Re-reading the bar in a kinder light is not a pass.
- Verify on data the change did not come from. In-domain numbers are reported as in-domain; a
  held-out split stays held out until the stage ends, then is scored once.
- Run the project's own checks — build, tests, lints, formatters — and report their actual output.
- Reproduce the headline number at least once more before quoting it anywhere.
- State the result plainly, including failure, regression, or no-change. A kill criterion that fires
  kills the attempt; it is not renegotiated.
- Separate "what was measured" from "what it means". Say which parts remain unverified.
- Then write it down: the project's log for reasoning and numbers, its status file for state, the
  commit message for the change.

**Artifact:** the verification result against the bar, and the updated log.

## 3. Loop closure

A task is not done at Beat 3. A phase is not done because its tasks are done — it is done when its
exit condition is measured. A stage is not done until its contract is settled either way, including
when the answer is that the approach does not work.

When a beat's artifact is missing, the work is not ahead of schedule; it is at an earlier beat than
it appears.

## 4. External-code questions: ask DeepWiki first (mandatory)

Any question about a repository you do not own — a dependency, a library, a framework, a paper's
reference implementation, a competitor's pipeline, a repo named in a prompt or a link — goes to the
**DeepWiki MCP server** before it is answered from memory and before its source is read by hand.
Without fail. This is the Beat 1 rule "read the primary source" made operational: DeepWiki is the
cheapest primary source for third-party code, and answering such a question from recollection is the
failure mode that rule exists to stop.

Server: `https://mcp.deepwiki.com/mcp` (HTTP transport, no auth required). Tools:

| Tool | Args | Use |
|---|---|---|
| `ask_wiki_question` | `repoName`, `question` | A specific question about behaviour, API, or internals |
| `read_wiki_structure` | `repoName` | What topics the wiki covers — use to scope a question |
| `read_wiki_contents` | `repoName` | The documentation itself, when the question is broad |

`repoName` is `owner/repo`, nothing else.

Rules:

- **Ask before you guess.** During Beat 1, and again before Beat 2 commits to anyone else's API or
  behaviour, ask. One `ask_wiki_question` call is cheaper than one wrong assumption.
- **Ask before reading the source by hand.** Use it to find *where* to look, then read that code.
  The two are complementary; skipping straight to a `grep` over an unfamiliar tree is not.
- **Answers are evidence, not ground truth.** DeepWiki indexes a snapshot and is AI-generated. Quote
  it with provenance, and confirm anything load-bearing — a signature, a default, a known bug —
  against the actual pinned version in `Cargo.lock` / lockfile / vendored source before building on
  it. A DeepWiki answer never satisfies Beat 4.
- **A miss is reported, not skipped.** If the server is unreachable, or has no wiki for the repo, say
  so explicitly and fall back to reading the source. Silently not asking is a protocol violation;
  asking and getting nothing is not.
- **Our own repos are exempt** — for those, the repo itself, its log, and its tests are the primary
  source.

## 5. Reporting

- Numbers carry their provenance: which machine, which data, which commit, how many runs.
- Selected or optimistic numbers are labelled as such, every time they are repeated.
- Never state a check passed without having run it. If a step was skipped, say which and why.
