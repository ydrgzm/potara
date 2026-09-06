# Fusion examples

These fusions use two widely installed skill sets:
[`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills) and
[`mattpocock/skills`](https://github.com/mattpocock/skills). Install those
first, then ask Potara to fuse the ones you want.

Each example describes what you get when the skills run one after another, and
what changes once they are fused.

## 1. Bug fixing loop (`potara` mode)

Fuse `diagnosing-bugs` (mattpocock), `debugging-and-error-recovery`
(addyosmani), and `tdd` (mattpocock).

```
"Fuse diagnosing-bugs, debugging-and-error-recovery and tdd, then fix this crash."
```

Run separately, these three overlap in an awkward way. Both
`debugging-and-error-recovery` and `diagnosing-bugs` start with a triage pass,
so the agent does that work twice. And `tdd` only enters after the bug is
already located, so the fix can land without a test that proves it is fixed.

Fused, the two triage passes become one. The step where `tdd` says "write the
failing test first" and the step where `diagnosing-bugs` says "set up feedback
that catches the bug" are the same step, so the merged procedure writes that
test once and keeps it as the regression test at the end. The order is:
reproduce, shrink the reproduction, write the failing test, find the cause, fix
until the test passes, keep the test. Where `tdd` asks for DAMP tests and
`debugging-and-error-recovery` just says "add tests", the more specific rule
wins.

You run one skill and the fix arrives with a regression test attached.

## 2. `specforge` (`dance` mode, writes a new skill)

Fuse `spec-driven-development` (addyosmani), `to-spec` (mattpocock), and
`to-tickets` (mattpocock).

```
"Make one skill out of spec-driven-development, to-spec and to-tickets. Call it specforge."
```

Run separately, there are two manual gaps. `to-spec` turns a conversation into a
spec but does no requirements gathering. `spec-driven-development` writes a full
PRD but assumes you already gathered the requirements. `to-tickets` turns a plan
into dependency-ordered tickets but needs a finished plan first.

Fused, `dance` writes `skills/specforge/SKILL.md` as one flow. It checks whether
the conversation already contains enough to synthesize a spec. If it does, it
takes the `to-spec` path. If it does not, it interviews first, until it is about
95% confident. Then it writes the PRD in the structure
`spec-driven-development` uses, and decomposes it into tracer-bullet tickets
with declared blocking dependencies. The templates from all three skills move
into the new folder.

One call takes you from a conversation to an ordered backlog, and the interview
step only runs when it is actually needed.

## 3. `implement` with built-in guardrails (`namek` mode)

Fuse `implement` (mattpocock, dominant) with `incremental-implementation`
(addyosmani) and `code-review-and-quality` (addyosmani).

```
"Have implement absorb incremental-implementation and code-review-and-quality."
```

`implement` on its own executes tickets and drives TDD at agreed points, then
finishes with a code review. Its slicing is loose, and its review is a single
pass. `incremental-implementation` adds thin vertical slices, feature flags, and
changes that are easy to roll back. `code-review-and-quality` adds a five-axis
review with severity labels and a target of about 100 lines per change.

Fused, `namek` rewrites `implement`'s `SKILL.md` in place and saves the old one
as `SKILL.original.md`. `implement` keeps its name, voice, and output format.
Its build step now carries the vertical-slice, feature-flag, and rollback
rules. Its review step now applies the five-axis rubric and the 100-line size
target as a gate before commit.

The skill you already trigger for implementation work now holds you to a slice
size and a review rubric without you loading anything else.

## 4. Frontend change, reviewed once (`potara` mode)

Fuse `frontend-ui-engineering`, `performance-optimization`, and
`security-and-hardening` (all addyosmani).

```
"Fuse frontend-ui-engineering, performance-optimization and security-and-hardening for this component."
```

Run separately, this is three review passes at three different times, and they
work against each other. A performance change can break an accessibility
contract. A new security header can block a fetch the component depends on.

Fused, it is one pass over one component. The agent builds the component with
the design-system, responsive, and accessibility rules, then applies
measure-first Core Web Vitals work and OWASP hardening to that same code. When
those three concerns pull in different directions, Potara's conflict rule says
to put the choice to the user rather than let whichever pass ran last decide.

You review once, and the trade-offs between accessibility, speed, and security
become explicit questions instead of silent overwrites.
