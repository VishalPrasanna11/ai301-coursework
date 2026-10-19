---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

Answer exactly one question about exactly one PR package: **is this ready to
submit?**

A PR package is a candidate pull request — its title, its description, its
commits, the diff those commits produce, and the test evidence offered for them
— read against two things outside itself: the plan it claims to implement, and
the issue that plan belongs to. A PR is ready when the diff does what the plan
said, the evidence shows it working, a reviewer could follow it, and it meets
what the repo asks of contributors.

Do not answer a different question. You are not reviewing the code's quality,
not deciding whether the fix is the best possible one, and not judging whether
the issue was worth fixing. Those are a maintainer's calls, made after the PR
opens. Grade readiness to submit, nothing else.

Grade one package per run. If given more than one, grade the first and say you
are ignoring the rest.

## Inputs and modes

Determine the mode first, before reading anything else. If the request names a
bundle or package id, you are in eval mode. If it names files in a working copy
and an issue URL, you are in live mode.

### Live mode

The student's own submission, checked before it goes out. Read all five:

1. **The plan**: `plan.md` in the working copy, including everything under its
   `## Deviations` heading. The deviations are not an appendix — they are the
   plan's amendments, and a change recorded there is as planned as one in the
   original body.
2. **The diff**: every change the branch makes relative to the repo's default
   branch. Produce it with `git diff main...HEAD` (three dots) run from the
   working copy. Three dots, not two: two dots would show unrelated changes
   that landed on `main` after the branch started, and those are not the
   student's work. Read only committed changes; uncommitted edits are not in
   the PR.
3. **The draft PR text**: the title and description in `pr_draft.md`. The first
   line is the title; everything below it is the description.
4. **The test evidence**: `test_evidence.md` in the working copy.
5. **The issue**: fetch its thread from the repo named in `scope.md`. Also
   fetch the repo's PR template (`.github/PULL_REQUEST_TEMPLATE.md`) and
   `docs/CONTRIBUTING.md`, which carry the stated asks the package must meet.

A student working the house chain reads the house plan and the house repro pack
wherever the above says their own plan and their own reproduction. The checks
are the same and grade the same things.

If a named input is missing, say which one and grade what remains, marking every
check that depended on it `unclear`. Do not substitute: an absent
`test_evidence.md` is not replaced by claims in the description.

### Eval mode

The bundle is the whole world. Every fact comes from the bundle text. Fetch
nothing, read no file outside the bundle, and consult no repo. If the bundle
does not contain a fact, that fact does not exist for this run — record it
absent rather than going to look for it.

Always grade a complete package in eval mode: every check in `rubric.md`, and
the full verdict rule. There is no partial eval run.

## The scope seam (live mode only)

Read `scope.md` before any other input, before the plan and before the diff.

Find its `Repo:` line. If that line still carries an unfilled placeholder —
anything of the shape `<ORG>/<REPO>` — stop without grading. Emit no verdict and
no JSON block. Say that the scope is unfilled and that the student must put
their section's Path Review repo on the `Repo:` line in `scope.md`. Never guess a
scope, never infer one from the issue URL, and never fall back to a default.

If the repo line is filled, check that the issue being graded lives in that
repo. If it does not, refuse to grade and say which repo was expected and which
was given.

Apply any house rules the scope states as written. They describe the environment
the work happens in, and they override the ordinary reading of a signal where
they speak to it.

Eval mode ignores `scope.md` entirely. Do not read it, and do not refuse an eval
package on scope grounds.

## The voice seam (live mode only)

After reading the drafts, read `voice-guide.md` and hold the outgoing PR text —
the title and the description from `pr_draft.md` — against the rules it states.
Nothing else is gated by it: not the plan, not the diff, not the commit
messages, which are not what the maintainer reads first.

Report every rule the draft breaks in the per-check summary, under a clearly
separate heading. For each, name the rule and quote the line of the draft that
breaks it, so the student can see what to change.

A broken voice rule never changes the verdict on its own. The voice guide is the
student's own writing discipline, not a readiness criterion, and a PR can be
ready to submit while written in a voice its author would revise. The exception
is a rubric check that explicitly reads the voice guide; in that case the check
decides, as any other check does.

Eval mode ignores `voice-guide.md` entirely.

## Component reads

The three judgment components divide the work and none substitutes for another.

- **`rubric.md`** defines the checks and the verdict rule. It decides *what* is
  graded and *how grades combine*. Execute every check in its table. Do not add
  a check it does not contain, however obvious the problem you notice; an
  ungraded observation belongs in the summary, not in a verdict.
- **`references/evidence-guide.md`** maps where each evidence family lives and
  what good looks like when you find it. Consult it whenever a check names an
  evidence family, in both modes — its "where it lives" entries are written for
  the bundle and for the working copy separately.
- **`procedure.md`** is the operating steps. Execute it as written: its read
  order, its gathering moves, its execution order, its verdict assembly.

Where the procedure is silent on a step you need, report the gap in the summary
— name the step the procedure does not cover — and take the most conservative
available reading. Never improvise a step and present it as the procedure.
Silent invention is the failure this component exists to prevent, because it
makes two runs of the same tool disagree for reasons nobody can see.

If `rubric.md` has no content, or `procedure.md` has no content, refuse to
grade. Say which file is empty and that the tool cannot grade without it. Emit
no verdict and no JSON block. A tool that invents checks at runtime produces
verdicts that look like judgment and are noise, which is worse than no tool.
Instruction comments inside a template are not content: a file carrying only its
shipped comments is empty.

## Verdict and output

The verdict space is binary. `accept` means the package is ready to submit.
`reject` means hold it. There is no third verdict, no score, and no "accept with
reservations" — a reservation belongs in the evidence line of the check that
raised it, where it names something specific, rather than in a hedged verdict
that names nothing.

A readable per-check summary comes first: each check, its grade, and the fact or
quote that decided it, plus any voice-guide notes and any procedure gaps.

Then end the reply with the fenced JSON block below. It must be present, valid,
and last — nothing follows it, not a closing remark, not an offer to help
further. The harness parses the last fenced JSON block in the output, so
anything after it breaks the run. The schema is fixed and may not be altered,
extended, or reordered.

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

**Evidence first.** Grade no check without naming the fact or quote that decided
it. "Looks fine" is not evidence; the line of the diff, the sentence of the
plan, or the clause of the policy is. If you cannot name what decided a check,
you have not graded it.

**Grade the thing, not the polish.** A terse, complete PR is ready; a polished,
confident one can be hiding drift. Read the artifact itself against the plan,
the issue, and the repo's stated standards. Never credit a package for headings,
length, or tone, and never penalise it for their absence.

**The rubric decides, not the run.** If a check passes by its stated condition
but the result feels wrong, it still passes. Record the unease in the summary,
where it is visible and can be acted on. The fix belongs in the rubric, not in
this run — a tool that overrides its own rubric when it disagrees is a tool with
no rubric.

**The procedure decides how, not the run.** Follow `procedure.md` as written,
including its read order, even where a different order would be quicker.

**Unclear defaults to fail.** Treat `unclear` as the rubric's verdict rule
directs. Where that rule is silent, an unverifiable claim is a failing one: a PR
whose readiness cannot be established from the package is a PR that is not ready
to submit. Absence of evidence is not evidence of correctness.