# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Posted upstream

**GitHub username**

VishalPrasanna11

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53#issuecomment-5989541973

Posting my plan for this, built from my reproduction above.

**Diagnosis.** All three separator positions in `PII_PATTERNS["phone_us"]` (`safety/pii_scrubber.py:16`) are `[-.]?` — dash or dot, no space. `(555) 123-4567` has a space after the closing parenthesis, so the pattern can't match it. The parentheses aren't the problem; the pattern already allows them, optionally. I isolated this by closing the space:

    >>> pat = PIIScrubber.PII_PATTERNS['phone_us']
    >>> re.search(pat, '(555) 123-4567')
    None
    >>> re.search(pat, '(555)123-4567')
    <re.Match object; span=(1, 13), match='555)123-4567'>

`detect()` and `scrub()` both iterate the same dict, which is why one cause produces both reported symptoms.

**Scope.** Two things.

1. One line in `safety/pii_scrubber.py`: add a literal space to the three separator classes, `[-.]?` → `[-. ]?`. A literal space rather than `\s` deliberately — `\s` matches newlines, which would let one "phone number" span two unrelated lines of the prose this scrubber runs over.

2. The four `strict=True` xfail markers, per `docs/CONTRIBUTING.md` ("removing the `@pytest.mark.xfail` line is part of fixing the issue"): lines 34, 46, 130, 191 of `tests/unit/test_pii_scrubber.py`. The suite should go from `20 passed, 5 xfailed` to `24 passed, 1 xfailed`.

Nothing else in `PII_PATTERNS`.

**Test plan.** Re-run the snippet from my repro (expecting both numbers redacted and `detect()` reporting the parenthesized one, with the dashed-only control unchanged), plus the pattern in isolation against the formats in `test_us_phone_formats` and two false-positive probes (`version 1.2.3 build 4567`, prose containing a bare number) to confirm the wider class hasn't started over-matching.

**Two things I want to flag rather than quietly handle:**

1. **The opening `(` survives.** With the fix, the match starts at index 1:

        >>> new = r'\b(?:\+?1[-. ]?)?\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})\b'
        >>> re.search(new, '(555) 123-4567')
        <re.Match object; span=(1, 14), match='555) 123-4567'>

    So `scrub()` returns `([REDACTED]` — no digits leak, but the paren stays. Pre-existing: the word-boundary assertion sits before the optional opening paren and can't match between the start of a string and `(`. It never surfaced because the space case never matched at all. Leaving it rather than moving that assertion, since that's a separate change to a separate part of the pattern. Happy to take it as a follow-up.

2. **`+1 555 123 4567` starts matching too**, where it didn't before:

        >>> re.search(pat, '+1 555 123 4567')    # current
        None
        >>> re.search(new, '+1 555 123 4567')    # with the fix
        <re.Match object; span=(1, 15), match='1 555 123 4567'>

    Same root cause, falls out of the same change, can't be excluded without special-casing. `test_us_phone_formats` already expects space-separated forms, so I'm treating it as intended rather than as extra scope.

**Not in scope: the marker on `test_mixed_pii_and_text` (line 237).** Its reason cites this issue, but it fails on `assert "Python" in scrubbed`, and the cause is `street_address`, not phones:

    >>> pat = PIIScrubber.PII_PATTERNS['street_address']
    >>> re.search(pat, '        I worked at TechCorp for 5 years developing Python applications.', flags=re.IGNORECASE)
    <re.Match object; span=(33, 63), match='5 years developing Python appl'>

The `Pl` alternative matches inside "applications", with `\b\d+\s+[A-Za-z\s]+` consuming the words in front of it. That's against the unmodified string, so it's independent of my change and of the order the patterns run in. CONTRIBUTING says to drop the marker from every test that covers the issue — this one is mislabelled rather than covered, so removing it would red the suite for a different bug. Leaving it and filing nothing for now; say the word if you'd rather I remove it and open a separate issue for the over-match.

---

## Your branch

**Branch**

`fix/53-parenthesized-phone-redaction`

**Evidence**

Environment for both runs: commit `f89c06f` (before) and `e5ecc12` (after), macOS
14.4.1, Python 3.12.4, pytest 9.1.1, `pip install -e ".[dev]"` in a venv.

### Before — the issue's snippet, as posted in my Unit 2 repro

    $ python -c "
    from safety.pii_scrubber import PIIScrubber
    s = PIIScrubber()
    for t in ['Call me at (555) 123-4567 or 555-123-4567', 'Call me at 555-123-4567']:
        print('in :', repr(t))
        print('out:', repr(s.scrub(t)))
        print('det:', s.detect(t))
    "

    in : 'Call me at (555) 123-4567 or 555-123-4567'
    out: 'Call me at (555) 123-4567 or [REDACTED]'
    det: [{'type': 'phone_us', 'value': '555-123-4567', 'start': 29, 'end': 41}]

    in : 'Call me at 555-123-4567'
    out: 'Call me at [REDACTED]'
    det: [{'type': 'phone_us', 'value': '555-123-4567', 'start': 11, 'end': 23}]

### Before — the suite

    $ python -m pytest tests/unit/test_pii_scrubber.py -v

    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_number_redaction XFAIL [ 12%]
    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_formats XFAIL [ 16%]
    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_phone_pii XFAIL [ 48%]
    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_phone_at_start_of_text XFAIL [ 72%]
    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_mixed_pii_and_text XFAIL [ 92%]

    ======================== 20 passed, 5 xfailed in 0.65s =========================

### After — the same snippet, same commands

    $ python -c "
    from safety.pii_scrubber import PIIScrubber
    s = PIIScrubber()
    for t in ['Call me at (555) 123-4567 or 555-123-4567', 'Call me at 555-123-4567']:
        print('in :', repr(t))
        print('out:', repr(s.scrub(t)))
        print('det:', s.detect(t))
    "

    in : 'Call me at (555) 123-4567 or 555-123-4567'
    out: 'Call me at ([REDACTED] or [REDACTED]'
    det: [{'type': 'phone_us', 'value': '555) 123-4567', 'start': 12, 'end': 25}, {'type': 'phone_us', 'value': '555-123-4567', 'start': 29, 'end': 41}]

    in : 'Call me at 555-123-4567'
    out: 'Call me at [REDACTED]'
    det: [{'type': 'phone_us', 'value': '555-123-4567', 'start': 11, 'end': 23}]

Both numbers are now redacted where only the dashed one was before, and
`detect()` returns two `phone_us` records where it returned one. The
dashed-only control line is byte-identical to the before run, including the
offsets, so the change did not disturb the format that already worked.

The `([REDACTED]` in the first output is the leftover opening paren my plan
named under Risks: the match starts at index 1, so the paren survives while no
digits leak. Predicted rather than discovered.

### After — the suite

    $ python -m pytest tests/unit/test_pii_scrubber.py -v

    tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_no_false_positives PASSED [100%]

    ======================== 24 passed, 1 xfailed in 0.15s =========================

The four phone tests pass with their markers deleted, and the single remaining
xfail is `test_mixed_pii_and_text`, left marked because its failure is the
`street_address` over-match rather than this bug. That is the exact figure the
plan predicted.

## Eval iterations

**Run history**

1. `--limit 3` smoke run — **3/3** (`clear-accept 2/2`, `wrong-cause 1/1`)
2. Full run, `--save-run eval-run.txt` — **20/20, PASS** (`clear-accept 7/7`, `scope-creep 4/4`, `thread-convention 2/2`, `unbuildable 3/3`, `wrong-cause 4/4`)

Run 2 is the committed `eval-run.txt`. No revision loop was needed: the smoke
run agreed on all three, and the full run agreed on all twenty with every
category matched, so there were no disagreements to feed back into the rubric.

**Package analysis**

**`pkg-04`**, in the `thread-convention` category. My rubric: **reject**. Gold
label: **reject**. They agree.

It is worth naming this one rather than a `clear-accept`, because the
`thread-convention` category is the floor case this week — only two packages
carry it, so a rubric blind to it fails the floor however well it does
elsewhere. My rubric caught it through `comment-answers-thread`, the one check
written specifically to read the plan comment against something outside the
plan: the thread's maintainer signals and the repo-facts block's stated
templates, contributing asks, and contribution policy.

The reason my rubric read it the way it did is structural rather than lucky.
Six of my seven required checks compare the plan against the repro evidence or
against itself — diagnosis against artifacts, approach against cause, test plan
against steps. All six can pass on a plan whose comment ignores what a
maintainer asked for, because none of them ever opens the thread. The seventh
is the only one whose evidence lives outside the plan document, and my
`procedure.md` enforces that by reading the repo-facts block third and the
thread highlights fourth, both before the candidate plan is read at all, so the
plan's framing cannot decide what counts as a convention. Without that read
order the check would still exist but would be grading the comment against
whatever the plan said the conventions were.

**Check rationale**

`comment-answers-thread`, quoted as it reads now in the `rubric.md` uploaded to
`tools/plan-check/`:

> The comment responds to what the thread and the repo actually ask. A
> maintainer's stated preference, pointer, or question about this issue is
> addressed rather than ignored. Where the repo's policy states a requirement —
> disclosing AI assistance, a template, an issue reference, a sign-off — the
> comment meets it; the absence of a denial is not a disclosure. Fails when the
> comment would read the same on any issue, or when a stated requirement goes
> unmet.

Two phrases in it are there because of specific failures I had already seen.

"The absence of a denial is not a disclosure" is carried over from the Unit 2
rubric, where the eval set contained exactly one disclosure-wall package: a
repo whose policy required disclosing AI assistance, with comments that were
silent on the question. Silence reads as compliance to a loose check, because
nothing in the comment contradicts the policy. Stating that absence fails is
what made that package gradable, and it is why this week's rubric cleared
`thread-convention 2/2` on a first run.

"Fails when the comment would read the same on any issue" is the test that
separates thread-aware from boilerplate, and it is deliberately about the
comment's content rather than its length or structure. The rubric template warns
that structure-shaped checks make graders disagree with themselves, so the
condition had to be something observable: strip the issue number and see whether
the comment still fits anywhere else.

The strongest evidence that the check is doing real work is that it caught me.
On my first live run against my own drafts, six required checks passed and this
one failed. `docs/CONTRIBUTING.md` says removing the `@pytest.mark.xfail` line
"is part of fixing the issue", and my draft comment said "Whether the markers
come off in this change is a maintainer call — I'd rather ask than delete them
unasked." That is a stated requirement being treated as optional, and the check
named it, quoting both the policy and my own sentence. I had written the plan
from what seemed polite rather than from what the repo documented. I revised
both drafts to remove the four markers as part of the fix, re-ran, and got
accept — and the build then confirmed the predicted `24 passed, 1 xfailed`.

**Trade-offs**

Nothing changed elsewhere, and here is how I know: the rubric that produced the
committed `eval-run.txt` is the rubric I wrote before the first smoke run.
Because both runs agreed on everything — 3/3 then 20/20 with all five
categories matched — there were no disagreements to revise away, so I never
loosened a check and never needed a canary. The eval-run header's fingerprints
for `rubric.md`, `procedure.md`, and `evidence-guide.md` are the ones I
uploaded, unmodified since.

What `comment-answers-thread` gives up is reach beyond what a repo writes down.
It grades the comment against stated requirements — a policy, a template, a
maintainer's comment in the thread — and has nothing to say about unstated
norms. A repo whose maintainers expect, by custom, that plans be discussed
before a branch is cut would have no artifact for this check to read, so a plan
comment violating that custom passes. The check is also only as good as its
source: on my own issue it reported no maintainer signals, correctly, because
every commenter on #53 is a classmate with association NONE. That is the right
answer here but it means the check contributes nothing on a thread with no
maintainer participation, which describes most of this course's issues. Its
value in that case rests entirely on the repo-facts half — the policy and
template side — which is exactly the half that caught my xfail mistake.

I accept that limitation rather than widening the check, because the
alternative is inferring conventions a repo has not stated, and a check that
guesses at norms is a check two people would grade differently.
