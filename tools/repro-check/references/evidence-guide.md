# Evidence guide: where proof lives in a reproduction package

## Environment

**Where it lives.** In an eval bundle: the repro report's own environment
section, checked against the issue context for any version the issue
targets, and against the repo-facts block for the documented setup path.
In live mode: the environment block of the draft comment, checked against
the issue body and the repo's setup docs.

**What good looks like.** The record names the build of the code under
test (commit, tag, or release), the language runtime version, and the OS.
Where the issue names a version or platform, the record either matches it
or says how it differs. Where the reporter skipped part of the repo's
documented setup, the report says which path was taken instead. The
observable condition: a reader could not run the same commands on a
different build and mistake it for this one.

## Steps

**Where it lives.** In an eval bundle: the command sequence in the repro
report, read from its first line as if from a clean checkout. In live
mode: the same sequence in the draft, read against the repo's setup docs
for anything assumed but not stated.

**What good looks like.** Every command between a clean checkout and the
shown output is present, in order, as text that can be copied and run.
Steps are given, not described — "installed the dependencies" is not a
step, `pip install -e ".[dev]"` is. Nothing depends on state the sequence
never establishes: no file that appears without being created, no service
assumed running, no environment variable used but never set.

## Behavior shown

**Where it lives.** In an eval bundle: the output excerpts, logs, and
test results in the repro report, read against the specific misbehavior
named in the issue context. In live mode: the pasted output in the draft,
read against the issue body.

**What good looks like.** The artifact shows the behavior the issue
reports. For a test cited as evidence, the failing assertion concerns the
thing the issue describes — a test that fails for an unrelated reason is
not evidence for this issue even when its marker or name suggests
otherwise. Where output is shown for a nearby defect, the report says so
rather than letting it stand as proof. The observable condition: a reader
comparing the artifact line by line against the issue's description finds
the same behavior, not a cousin of it.

## Honesty

**Where it lives.** In an eval bundle: the report's stated conclusion,
read against the artifacts in the same report. In live mode: the draft's
summary lines, read against what its own pasted output shows.

**What good looks like.** Every assertion is backed by something shown.
A reproduction is claimed only where an artifact demonstrates it. A
failed attempt is reported as a failed attempt, with the evidence of what
was tried — a well-evidenced cannot-reproduce is a complete and passing
report, not a deficient one. A cause is asserted only where it was tested
in isolation; where it was inferred from reading, it is marked as a
guess. The failure mode to catch: a confident conclusion the artifacts
underneath do not support.

## Comms

**Where it lives.** In an eval bundle: the claim comment and repro report
text, read against the repo-facts block's stated bug-report template asks
and contribution policy, including any AI-use policy. In live mode: the
draft comments, read against the repo's CONTRIBUTING file, issue
templates, and any stated policy on AI assistance.

**What good looks like.** Where the repo's policy states a requirement,
the comment text meets it. A policy requiring disclosure of AI assistance
is met only by an actual disclosure in the comment — the absence of a
denial is not a disclosure, and a comment that is silent on the question
fails against such a policy however good its evidence. A required
template, issue reference, or sign-off is present when asked for. Beyond
compliance: the claim commits to investigation rather than to an outcome
or a date, and the words are specific to this issue rather than
boilerplate that would fit any thread.