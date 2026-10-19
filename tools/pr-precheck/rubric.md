# Rubric: is this pull request ready to submit?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diff-within-plan | The diff's changed hunks, read against the plan's stated scope and everything under its `## Deviations` heading | Every hunk traces to something the plan authorises: its in-scope statement, or a deviation note recording what changed and why. A change the plan never named fails, unless a deviation note covers it. Size never decides this — a plan authorising ten files passes with ten. Fails when the diff does something the plan and its deviations are both silent about | required |
| plan-work-accounted | The plan's in-scope items, read against the diff and against the description's own statements about what was left out | Work the plan promised is either present in the diff, or named as not done — in a deviation note, or in the description. A package that quietly delivers less than the plan fails. A package that delivers less and says so, with the reason, passes: an honestly disclosed shortfall is ready to submit | required |
| description-matches-diff | The description's Summary and Changes sections, read line by line against the diff | Every change the description claims appears in the diff, and every file the diff touches is accounted for by some line of the description. Fails when the description asserts a change the diff does not contain, or omits a changed file a reviewer would be surprised by | required |
| evidence-observable | The test evidence, read against the plan's test plan and the behaviour the issue describes | The evidence shows a specific observation that differs before and after, reachable by commands the evidence gives. A before/after pair of command output satisfies this. Fails when success is asserted without output, when only one side is shown so nothing can be compared, or when the output shown does not bear on the issue's behaviour | required |
| repo-checks-run | The test evidence, read against the checks the repo's PR template or contributing docs name | Each check the repo asks contributors to run appears in the evidence with its actual result. A check that fails counts as run and reported, provided the evidence says it failed and why; a check whose result is absent or summarised without output does not. Fails when a required check is missing from the evidence, or when a failure is stated as a pass | required |
| no-debris | The diff's changed hunks | No hunk is debris: no commented-out code, no debugging or print statements left behind, no stray whitespace-only changes, no files committed that are working notes rather than part of the fix. Fails on any hunk a reviewer would ask to be removed before merging | required |
| no-unrelated-hunks | The diff's changed hunks, read against the issue's subject | Every hunk serves this issue's fix. An improvement, cleanup, rename, or reformat the issue did not call for fails here even when it is an improvement and even when the description explains it, because it costs a reviewer attention the fix needed. A mechanical change the fix forces — a lint fix on a line the fix touched — passes | required |
| template-sections-filled | The description, read against the repo's stated PR template | Every section the template asks for carries a real answer. A section answered "N/A" passes when the reason is given. Fails when a template section is missing, or left with its placeholder text, or ticked as done where the evidence shows otherwise | required |
| policy-asks-met | The repo's stated contribution policy and PR template, read against the description and the diff | Where the repo states a requirement of contributors, the package meets it. A policy requiring disclosure of AI assistance is met only by an actual disclosure in the description: silence is not a disclosure, and the absence of a denial is not one either. Where the policy requires an issue reference, a sign-off, a marker removal, or a convention, the package carries it. Fails when a stated requirement goes unmet | required |
| issue-linked | The description, read against the issue the package claims to implement | The description names the issue it closes, by number, in a form a reader can follow. Fails when no issue is named, or when the number named is not the issue the plan belongs to | required |
| title-specific | The PR title | The title names what changed, specifically enough that someone scanning a list of PRs could tell this one apart. Fails on a title that would fit any change to this repo | preferred |
| commits-readable | The commits, read as a sequence | The commits divide the change into steps a reviewer could read in order, rather than one undifferentiated commit or a trail of fixups | preferred |

## Verdict rule

Accept (ready to submit) if and only if every `required` check passes. A single
required fail holds the package.

`unclear` counts as fail on a required check. A package whose readiness cannot
be established from what it contains is not ready to submit: absence of evidence
is not evidence of correctness, and a reviewer cannot verify an intention the
author did not write down.

`preferred` checks never change the verdict. They rank readiness among packages
that already pass, with `title-specific` weighted above `commits-readable`.

**The honest-outcome rule applies across every required check.** A package that
does less than the plan intended, or whose checks did not all pass, or whose fix
has a known limitation, is still ready to submit when it says so plainly and
gives the reason. What fails is the gap between what a package claims and what
it contains — not the shortfall itself. Judge each check against that gap.