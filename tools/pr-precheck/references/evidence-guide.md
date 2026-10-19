# Evidence guide: where evidence lives in a PR package

## Plan fidelity

**Where it lives.** In an eval bundle: the candidate plan section, including any
deviation notes, read against the diff section's changed hunks. In live mode:
`plan.md` in the working copy, everything under its `## Deviations` heading
included, read against `git diff main...HEAD` from the same working copy. A
house-chain student's plan is the house plan.

Two facts have to be assembled before this family can be graded, and neither is
in one place: the plan's authorised set (its in-scope statement plus every
deviation note) and the diff's actual set (every changed hunk). The comparison
runs both ways.

**What good looks like.** Every hunk traces to an authorising line of the plan
or a deviation note, and every in-scope plan item is either in the diff or named
as not done. A deviation note is as authoritative as the plan's body — it is the
plan amended, not an excuse appended. The observable condition: reading the plan
and then the diff, a reviewer finds no change they were not told about and no
promise they cannot locate or account for. A package delivering less than the
plan is still faithful when the shortfall is named with its reason; what breaks
fidelity is a diff that quietly does more or less than the plan said.

## Test evidence

**Where it lives.** In an eval bundle: the test evidence section, read against
the plan's test plan and the issue context's described behaviour. In live mode:
`test_evidence.md` in the working copy, read against `plan.md`'s test plan and
the issue thread. The repo's named checks come from the PR template and
`docs/CONTRIBUTING.md`.

**What good looks like.** A before and an after, both as command output rather
than description, with the command that produced each. The observation that
differs between them bears on the behaviour the issue reports — a test moving
from failing to passing, an output string appearing, a count changing. Each
check the repo asks contributors to run appears with its actual result pasted,
not summarised. A check that failed is still evidence when the evidence says it
failed and why; a check whose output is absent is not evidence at all,
regardless of what the description claims about it. The failure mode to catch:
evidence that proves nothing observable, because only one side is shown, or
because the output does not touch the reported behaviour.

## Diff quality

**Where it lives.** In an eval bundle: the diff section, read hunk by hunk
against the issue context's subject. In live mode: `git diff main...HEAD` from
the working copy — three dots, so that changes landing on `main` after the
branch started are excluded — read against the issue. `git diff main...HEAD
--stat` gives the file list to check first.

**What good looks like.** Every hunk serves this issue's fix and nothing else.
Debris is anything a reviewer would ask to have removed before merging:
commented-out code, debug or print statements, whitespace-only changes, working
notes committed alongside the fix. Separately from debris, an unrelated hunk is
a change that works but that this issue did not call for — a cleanup, a rename,
a reformat of untouched code. Both cost a reviewer the attention the fix needed,
which is why an unrelated improvement fails this family even when the
description explains it. A mechanical change the fix forces, such as a lint fix
on a line the fix already touched, is part of the fix. The observable condition:
a reviewer reading the diff cold asks no question of the form "why is this
here?"

## Standards and comms

**Where it lives.** In an eval bundle: the repo-facts block's template asks and
stated contribution policy, read against the candidate PR's title and
description. In live mode: the repo's `.github/PULL_REQUEST_TEMPLATE.md` and
`docs/CONTRIBUTING.md`, read against `pr_draft.md` — first line the title, the
rest the description — and against the issue thread for anything a maintainer
asked for there.

**What good looks like.** Every section the template asks for carries a real
answer; "N/A" with a reason is a real answer, and placeholder text left in place
is not. Every requirement the policy states is met in the description. A policy
requiring disclosure of AI assistance is met only by an actual disclosing
sentence — silence is not a disclosure, the absence of a denial is not a
disclosure, and a description that is otherwise excellent fails this family when
it is silent on a disclosure the repo requires. Where the policy requires an
issue reference, a sign-off, a convention, or a marker removal, the package
carries it. The issue is named by a number a reader can follow, and it is the
issue the plan belongs to. Beyond compliance: the title names what changed
specifically enough to tell this PR apart in a list, and the description would
not fit unchanged on any other PR to this repo.