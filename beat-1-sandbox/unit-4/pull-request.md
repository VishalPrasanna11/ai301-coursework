# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/229

**Branch**

`fix/53-parenthesized-phone-redaction`

## Eval iterations

**Run history**

1. `--limit 3` smoke run — **3/3** (`clear-accept 1/1`, `silent-drift 1/1`, `standards-wall 1/1`)
2. Full run, `--save-run eval-run.txt` — **18/20, PASS** (`clear-accept 5/7`, `not-tested 4/4`, `silent-drift 4/4`, `standards-wall 2/2`, `unreviewable 3/3`)

Run 2 is the committed `eval-run.txt`. No revision loop followed it. The run
cleared the bar at exactly 18 with every category matched, including
`standards-wall`, the two-package floor category. The two misses are both the
same check failing the same way, described below; I chose to leave the rubric as
the run graded it rather than spend a second full run chasing 20, since the
score sets no points and the miss is the most useful thing I learned this week.

**Package analysis**

**`pkg-05`** (nushell, keybinding merge-by-name dropping bindings that share a
name across different keys), in the `clear-accept` category. My rubric:
**reject**. Gold label: **accept**.

It failed exactly one check, `repo-checks-run`. The package's repo-facts block
says the contributing guide names `cargo fmt`, `clippy`, and `cargo test`, and
the candidate PR's evidence ends:

> `cargo test -p nu-protocol` passes (312 tests); fmt and clippy clean.

All three named checks are reported. One carries a test count. None carries
pasted terminal output.

My check demanded that each named check appear "with its actual result", and
then spelled out what that excluded: "a check whose result is absent or
summarised without output does not" count. "fmt and clippy clean" is precisely a
summary without output, so the check failed, and because it is `required` with
`unclear` and `fail` both holding the package, the verdict went to reject.

The distinction I had collapsed is between a passing check and a failing one.
For a failure, the output is doing real work: it carries the reason, and a
one-line summary can hide whether the author understood what broke. For a pass,
"clippy clean" is already complete information — there is no further fact a
pasted terminal line would add, because a passing linter has nothing to say. My
pass condition treated both cases identically and so demanded ceremony from the
honest case. `pkg-08`, the other miss, failed the same check for the same
reason.

What makes this worth writing about rather than quietly fixing is that it is the
same error in a different costume from the one my Unit 3 rubric made: a check
written for one shape of artifact, applied to a shape it did not anticipate.
There it was `environment-recorded` demanding a language runtime version from a
compiled CLI tool that has none. Here it is `repo-checks-run` demanding output
from a check that produces none worth printing. Both times the fix is the same
move — make the condition about what a reader needs to know, not about the form
the information arrives in.

**Check rationale**

`repo-checks-run`, quoted as it reads now in the `rubric.md` uploaded to
`tools/pr-precheck/`:

> Each check the repo asks contributors to run appears in the evidence with its
> actual result. A check that fails counts as run and reported, provided the
> evidence says it failed and why; a check whose result is absent or summarised
> without output does not. Fails when a required check is missing from the
> evidence, or when a failure is stated as a pass.

The clause I would now change is the one that cost me two packages, and I want
to be precise about which half of this check is right and which half is wrong,
because they came from different places.

The first sentence of the failure clause — "a check that fails counts as run and
reported, provided the evidence says it failed and why" — is the honest-outcome
rule, carried forward from three previous weeks and stated in this rubric's
verdict rule as applying across every required check. It is the reason this
check does not punish a red suite, and it is why my own package passes it
despite `make test-unit` failing. That half I would keep unchanged.

The second half — "or summarised without output does not" — is where I
overreached. I wrote it to catch the `not-tested` category, where packages claim
success with nothing behind the claim, and it does catch them: `not-tested 4/4`
on the full run. But "summarised without output" is a statement about the form
of the evidence rather than about whether a reader can tell what happened, and
the rubric template warns in as many words that structure-shaped conditions are
what make graders disagree with themselves. A reader learns nothing more from a
pasted `clippy` invocation than from the sentence "clippy clean". What they need
output for is a failure.

What I would write instead, and did not have the runs left to confirm: require
pasted output only where a check failed or where its result bears on the
reported behaviour, and accept a stated result for a check that simply passed.

**Trade-offs**

What this check gives up in its current form is two clear-accepts, and I know
exactly which two and exactly why: `pkg-05` and `pkg-08`, both reporting every
named check, both summarising a passing one instead of pasting it. That is the
check being too strict in the direction that costs agreement on the largest
category — `clear-accept` has seven packages, more than any other, so a check
that reads honest brevity as insufficient evidence is expensive.

Nothing else changed, and here is how I know: the rubric, procedure, evidence
guide, and SKILL.md that produced the committed `eval-run.txt` are the ones I
wrote before the smoke run, unmodified since. Because the smoke run agreed 3/3
and the full run agreed 18/20, I never loosened a check, so there was no
revision whose blast radius needed a canary. The eval-run header's fingerprints
for those files match the copies uploaded to `tools/pr-precheck/`.

The case I accept the check will still miss, even loosened: a package that
pastes real output for a check it ran against the wrong target — the right
command in the wrong directory, or on the wrong branch. The output would look
exactly like evidence and the check reads only whether output is present and
what result it states, not whether the run it came from was the run that
mattered. Catching that would mean verifying the provenance of each pasted
block, which the package does not carry enough information to support.

Running my own PR through the finished tool is what convinced me the trade-off
sits in the right place overall. `repo-checks-run` failed my package too, and
correctly: CONTRIBUTING names five local checks and my evidence covered three,
because I had worked from the PR template's list instead of the contributing
guide's. The check that is too strict on the eval set found a real gap in my own
submission on the first run — `make format` and the frontend tests, both of
which I then ran. The same strictness that costs two packages bought that, and I
would rather it erred this way than let a missing check through.
