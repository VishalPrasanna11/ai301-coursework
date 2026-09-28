# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| environment-recorded | The repro report's environment record, read against the issue context for any version the issue targets and against the repo-facts block for the documented setup path | The record identifies the build of the software under test — a commit, tag, or release version — together with whatever else a reader would need to get the same build, such as the OS or install method where those could change the outcome. A runtime or dependency version is required only where the software under test has one that could change the behavior. Where the issue targets a different version, the report says so. Where the reporter skipped part of the documented setup, the report says which path was taken instead. Fails only when a reader could not tell what was run, or could reasonably get a different build without knowing it | required |
| steps-executable | The commands and setup actions in the report, read as a sequence a stranger would follow to reach the shown output | A reader can get from nothing to the shown output by following the report. Commands that produce the output are given as runnable text. Setup that is not a command — creating a fixture file, editing a config — may be described, provided the description is precise enough to perform without guessing, including by pointing at exact content already present in the issue. Fails when a needed step is missing, when a description leaves a reader guessing at content or values, or when the sequence depends on state it never establishes | required |
| behavior-matches-issue | The output excerpts read against the specific behavior the issue describes | The artifact shows the behavior the issue reports, not an adjacent one. Where the package attributes a failure to the issue, the failing assertion must be about the behavior the issue names. Fails when output is shown for a different defect, or when a test is claimed as evidence but its assertion concerns something the issue does not describe | required |
| outcome-stated-honestly | The report's stated conclusion, read against its own artifacts | The conclusion matches the evidence shown. A reproduction is claimed only where the artifact shows it; an unreproducible attempt is reported as such with the evidence of the attempt; a cause is asserted only where it was tested in isolation, and otherwise marked as a guess. A well-evidenced cannot-reproduce passes. Fails on a confident claim the artifacts do not support | required |
| conventions-respected | The repo's stated policy in the repo-facts block, read against the text of the claim and repro comments | The comments comply with the repo's stated requirements for contributors. Where the policy requires disclosing AI assistance, the comments disclose it; where the policy requires a template, issue reference, or sign-off, the comments carry it. Fails when the policy states a requirement and the comment text does not meet it | required |
| claim-promises-only | The claim comment's forward-looking statements | The claim commits to investigation and nothing more. No fix promise, no delivery date, no assertion of a cause not yet tested. Fails on any promised outcome or deadline | required |
| control-shown | The report's artifacts, looking for a working comparison case | The report shows an adjacent case that behaves correctly, isolating the defect rather than leaving open whether the whole feature is broken | preferred |
| scope-separated | The report's treatment of anything it found beyond the issue | Where the investigation surfaced a defect other than the one reported, the report names it as out of scope rather than folding it into this issue's evidence | preferred |

## Verdict rule

Accept (ready to post) if and only if every `required` check passes. A
single required fail holds the package.

`unclear` counts as fail on a required check: a package whose readiness
cannot be established from the evidence in it is not ready to post.

`preferred` checks never change the verdict. They rank readiness among
packages that already pass, with `control-shown` weighted above
`scope-separated`.