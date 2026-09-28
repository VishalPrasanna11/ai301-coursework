# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

## Selected issue

### Issue link

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53

### Verdict output

```
Summary

All three issues come from the correct scope (codepath/pathreview-ai301-fa26-s1) and share identical repo-level facts: last push 2026-09-16 (7 days before today, well within 90), commits authored by a human maintainer (Andrew Burke), and a CONTRIBUTING.md that welcomes fork+PR contributions with no AI ban. Each issue is an unclaimed, good first issue-labeled bug with a specific trigger/misbehavior, an empty (hence unresolved-question-free) comment thread, no assignee, and no linked PR, and each names failing unit tests in a file confirmed to exist. All three accept.

Ranked by fit (Python application code, testable, unambiguous target behavior):

1. #53 — PII scrubber misses parenthesized phone numbers — cleanest fit: a regex/pattern fix in safety/pii_scrubber.py with an unambiguous correct outcome (the number gets redacted) and 4 named tests to verify it.
2. #57 — Tech detector counts vendored/build files — equally well-scoped path-filtering fix in agent/tools/tech_detector.py, with 2 named tests.
3. #56 — Structural chunker drops headingless docs — accepted, but the fix has more design latitude (issue floats two possible remedies — single-block chunk vs. fallback strategy — rather than one obvious behavior), so it costs a bit more judgment than the other two before the one named test tells you you're right.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "last push 2026-09-16T21:48:27Z, 7 days before capture date 2026-09-23"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits all authored by human Andrew Burke (Aburke225)"},
      {"name": "policy-permits-contribution", "grade": "pass", "evidence": "docs/CONTRIBUTING.md: 'Fork the repository and clone your fork' ... 'Comment on the issue' — standard open PR workflow, no AI-contribution ban found in CONTRIBUTING.md or elsewhere in repo"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "carries maintainer-applied 'good first issue' label, plus is a defect with a specific misbehavior (parenthesized format not matched)"},
      {"name": "behavior-specified", "grade": "pass", "evidence": "body gives exact trigger and misbehavior: '(555) 123-4567... passes through scrub() unredacted, and detect() reports no PII for it' with runnable repro"},
      {"name": "no-open-design-question", "grade": "pass", "evidence": "comments: [] — empty thread"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: [], no linked or cross-referenced PRs found via GraphQL timeline query"},
      {"name": "newcomer-labeled", "grade": "pass", "evidence": "labels include 'good first issue'"},
      {"name": "test-surface", "grade": "pass", "evidence": "issue names 4 tests in tests/unit/test_pii_scrubber.py, file confirmed to exist in repo"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/57",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "last push 2026-09-16T21:48:27Z, within 90 days of 2026-09-23"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits all authored by human Andrew Burke (Aburke225)"},
      {"name": "policy-permits-contribution", "grade": "pass", "evidence": "CONTRIBUTING.md permits fork+PR contributions; no AI ban or unmeetable precondition found"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "carries 'good first issue' label; defect has specific misbehavior (vendored/build paths counted)"},
      {"name": "behavior-specified", "grade": "pass", "evidence": "body states exact trigger and expected vs. observed: primary_language reported 'JavaScript' but 'expected: Python' given the file mix"},
      {"name": "no-open-design-question", "grade": "pass", "evidence": "comments: [] — empty thread"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: [], no linked/cross-referenced PRs"},
      {"name": "newcomer-labeled", "grade": "pass", "evidence": "labels include 'good first issue'"},
      {"name": "test-surface", "grade": "pass", "evidence": "issue names 2 tests in tests/unit/test_tech_detector.py, file confirmed to exist"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/56",
    "checks": [
      {"name": "repo-alive", "grade": "pass", "evidence": "last push 2026-09-16T21:48:27Z, within 90 days of 2026-09-23"},
      {"name": "maintainer-present", "grade": "pass", "evidence": "last 5 default-branch commits all authored by human Andrew Burke (Aburke225)"},
      {"name": "policy-permits-contribution", "grade": "pass", "evidence": "CONTRIBUTING.md permits fork+PR contributions; no AI ban or unmeetable precondition found"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "carries 'good first issue' label; defect has specific misbehavior (empty list for headingless docs)"},
      {"name": "behavior-specified", "grade": "pass", "evidence": "body states trigger and misbehavior: 'chunk() returns an empty list for any document without markdown headings' with runnable repro showing 0 chunks"},
      {"name": "no-open-design-question", "grade": "pass", "evidence": "comments: [] — empty thread"},
      {"name": "unclaimed", "grade": "pass", "evidence": "assignees: [], no linked/cross-referenced PRs"},
      {"name": "newcomer-labeled", "grade": "pass", "evidence": "labels include 'good first issue'"},
      {"name": "test-surface", "grade": "pass", "evidence": "issue names 1 test in tests/unit/test_structural_chunker.py, file confirmed to exist"}
    ],
    "verdict": "accept"
  }
]
```

## Eval iterations

### Run history

1. `--limit 3` smoke run — **2/3**
2. `--limit 3` after revising `maintainer-present` and `scope-bounded` — **3/3**
3. Full run, `--save-run eval-run.txt` — **16/20** (below the bar; categories `claimed 4/4  clear-accept 5/8  dead-repo 3/3  policy 1/1  scope 3/4`)
4. `--only issue-04,issue-15,issue-16,issue-19` after rewriting `scope-bounded` by issue type and loosening `behavior-specified` — **2/4**
5. `--only issue-04,issue-19` after restructuring `scope-bounded` into three pass routes — **2/2**
6. Confirming full run, `--save-run eval-run.txt` — **19/20, PASS** (categories `claimed 4/4  clear-accept 7/8  dead-repo 3/3  policy 1/1  scope 4/4`)

Run 6 is the committed `eval-run.txt`.

### Issue analysis

**`issue-04`** (zxcalc/zxlive#555, "Missing several basic rule previews").
My rubric: **reject**. Gold label: **accept**.

The whole body is one line:

> Including remove identity, fuse spiders, remove self loops, etc.

My `scope-bounded` check failed it, and on the body alone it was applying my
rule correctly. There is no enumeration of which previews are missing, no
reproduction, and an "etc." that leaves the extent literally unbounded. My check
asked whether a newcomer could tell when they were done, and from that sentence
the answer is no.

What my rubric could not see is what the gold label is reading. The issue's
label line is:

> labels: Type: bug, good first issue, Category: Proof mode, Priority: Medium

`good first issue` was applied by RazinShaikh, listed in the bundle as
`COLLABORATOR` and the author of most of the repo's recent commits. On a project
with 64 open items, a maintainer marking something beginner-appropriate is a
judgment about size from someone who knows where the code lives. The terse body
is not evidence of an unbounded task; it is evidence that the task is obvious to
anyone who has the repo open.

So the disagreement was not a threshold being slightly off. My rubric was reading
one source of boundedness — the issue text — when there are two, and on a terse
issue the second is the more reliable one.

### Check rationale

`scope-bounded`, quoted as currently written in the `rubric.md` uploaded to
`tools/issue-select/`:

> Pass if any one of these holds. (a) A `good first issue`, `beginner`, or
> equivalent label was applied by a maintainer or collaborator: that is a
> maintainer's judgment that the work is appropriately sized, and a terse body
> does not fail an issue a maintainer has vouched for. (b) The issue is a defect
> whose misbehavior is specific, even with no fix prescribed: the bug report
> defines the finish line, and a diagnosis of the cause strengthens it. Optional
> extras explicitly marked as suggestions, nice-to-haves, or future work are
> excluded from the extent and never fail this check. (c) The issue is a docs or
> feature request whose changes are enumerated specifically enough to know when
> they are done. Size alone never fails. Fail only when none of these holds,
> meaning the extent is open-ended and the contributor must first decide whether
> or how to do it

It reached this form over three versions. It started as a single condition about
localized edits, which conflated how *big* a change is with whether its *extent
is settled* before you start. Those come apart in both directions: `issue-01` is
a large docs change (a new page plus edits to four files) that is safe because
every piece is spelled out, while `issue-04` is a one-line change that is unsafe
on its face because nobody has said where it stops. Size was the wrong axis, so
"Size alone never fails" is now stated outright to stop the check drifting back
toward it.

The three numbered routes exist because boundedness arrives by three different
mechanisms. A defect is bounded by its own misbehavior — the bug report defines
the finish line, so no fix needs to be prescribed. A feature or docs request has
no equivalent natural boundary and must be enumerated. A maintainer's label is
external evidence that outweighs a thin body. Earlier versions buried these as
qualifying clauses inside one paragraph and the check kept failing the very
issues those clauses were written to protect: the optional-extras sentence was
already present in the version that still rejected `issue-19`. Promoting them to
numbered alternatives is what made the check behave.

### Trade-offs

Route (a) is what this check gives up. It lets an issue pass on a
`good first issue` label alone, with no enumeration and no reproduction in the
body — which is exactly what `issue-04` is. I am trusting a maintainer's tag over
the evidence in front of me, and labels go stale, get applied optimistically, and
on some repos are sprayed across anything that looks small.

The case I accept it will miss: an issue labelled beginner-friendly months ago
that has since accumulated scope in its comment thread, or one labelled by a
maintainer who was guessing. My check reads the label as present or absent and
never asks how old it is or who applied it relative to the issue's current state.

I also re-ran `--only issue-04,issue-19` as a canary after adding route (a),
specifically to confirm the loosening had not simply made the check pass
everything: both went from reject to accept as intended, and the following full
run held at `scope 4/4`, so the scope category did not degrade elsewhere.

## Selection rationale

**1. Fit to my interests and the time available.** #53 is a pattern-matching fix
in `safety/pii_scrubber.py` — parenthesized US phone numbers like `(555) 123-4567`
pass through unredacted. Python is one of the two languages I have actually
written non-trivial code in, and a regex correction against four existing tests is
about the smallest possible unit of real contribution. That matters because I am
already behind on this unit; I wanted an issue where the time goes into
understanding the codebase rather than into fighting an unfamiliar toolchain.

**2. What the verdict identified, and what I weighed that it could not.** The
skill correctly established the mechanical facts: the repo is live (last push
seven days before the run), the issue is unclaimed with no linked PRs, the
contribution policy permits a fork-and-PR, and `tests/unit/test_pii_scrubber.py`
exists with the four tests the issue names. It ranked #53 above #56 and #57 on
fit. What it could not weigh is that #53's correct end state is not open to
interpretation — either the number is redacted or it is not — whereas #56 floats
two possible remedies in its body without settling on one. #56 passed
`scope-bounded` only through route (a), the maintainer label, rather than through
route (b). A check passing on its weakest route told me the issue costs more
judgment than its `tier-1` tag suggests, and that is a distinction my rubric
records but does not act on.

**3. Anticipated difficulty in claiming it.** The house rule in my `scope.md`
says classmates' claim comments do not block an issue here, so the main risk is
not being beaten to it — it is that #53 is an obvious pick (tier-1,
`good first issue`, no comments at the time I graded it) and several people may
land on the same fix. Since credit attaches to the pull request I open rather
than to whether it merges, I am treating that as acceptable. The real work in
Unit 2 will be reproducing the failure locally: the issue names the tests but I
have not yet set up the repo, so my first task is confirming those four tests
actually fail before I touch the regex.
