# Plan: fix #53 — PII scrubber fails to redact parenthesized US phone numbers

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53
Reproduction: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53#issuecomment-5865057790
Branch: `fix/53-parenthesized-phone-redaction`

## Diagnosis

The `phone_us` pattern admits a dash or a dot between number groups, but not a
space. `(555) 123-4567` has a space after the closing parenthesis, so the
pattern cannot match it.

From `safety/pii_scrubber.py` line 16, all three separator positions are
`[-.]?` — dash or dot, optional. No position accepts a space.

I tested this in isolation rather than reading it off the pattern. From my
posted reproduction:

    '555-123-4567'   -> <re.Match object; span=(0, 12), match='555-123-4567'>
    '(555) 123-4567' -> None
    '(555)123-4567'  -> <re.Match object; span=(1, 13), match='555)123-4567'>
    '555.123.4567'   -> <re.Match object; span=(0, 12), match='555.123.4567'>

The third line is the control that identifies the cause: closing the space in
the parenthesized form restores the match. So the parentheses are not the
problem — the pattern already allows them, optionally. The space is.

That also explains the `detect()` behavior in the same report: `detect()` and
`scrub()` both iterate the same `PII_PATTERNS` dict, so a pattern that does not
match produces neither a redaction nor a detection record. One cause, both
symptoms.

## Scope

Two things, because `docs/CONTRIBUTING.md` makes the second part of the fix
rather than an option: "removing the `@pytest.mark.xfail` line is part of
fixing the issue. Your PR for issue #56 should both fix the bug and drop the
marker from every test that covers it." (#56 is the doc's own example issue;
the rule is general.)

**In scope.**

1. The separator character class in the `phone_us` pattern,
   `safety/pii_scrubber.py` line 16. One line.
2. The `@pytest.mark.xfail` markers on the four tests that cover this bug, in
   `tests/unit/test_pii_scrubber.py`: `test_us_phone_number_redaction`
   (marker at line 34), `test_us_phone_formats` (46), `test_detect_phone_pii`
   (130), and `test_phone_at_start_of_text` (191). All four get deleted. The
   markers are `strict=True`, so leaving one on a now-passing test fails CI
   with `XPASS(strict)`.

**Not in scope.**

- **The marker on `test_mixed_pii_and_text` (line 237).** Its reason string
  cites #53 like the other four, but the test does not cover this bug. It fails
  on `assert "Python" in scrubbed`, because
  `PII_PATTERNS["street_address"]` matches the span
  `5 years developing Python appl` — the `Pl` alternative matching inside
  "applications", with `\b\d+\s+[A-Za-z\s]+` consuming the words in front of
  it. I confirmed that against the unmodified string, so it is independent of
  my change and of the order the patterns are applied in. CONTRIBUTING says to
  drop the marker from every test that covers the issue; this one is
  mislabelled rather than covered, so removing its marker would turn the suite
  red for a defect I am not fixing. It stays, and the `street_address`
  over-match belongs to a separate issue.
- **The leftover `(`** described under Risks below. A pre-existing property of
  this pattern, not something this change introduces.
- **Any other pattern in `PII_PATTERNS`,** and the `E501` suppression for this
  file in `pyproject.toml` — that one names a formatting constraint (a long
  inline regex) rather than a seeded defect, and the line stays long either
  way.

## Files to touch

- `safety/pii_scrubber.py`, line 16, the `phone_us` entry of `PII_PATTERNS`.
- `tests/unit/test_pii_scrubber.py`, deleting four `@pytest.mark.xfail`
  decorators (a four-line block at each of 34, 46, 130, and 191).

No other file.

## Approach

Add a literal space to each of the three separator character classes, so each
`[-.]?` becomes `[-. ]?`. Then delete the four marker blocks named above.

A literal space rather than `\s`, deliberately. `\s` also matches newlines and
tabs, which would let a single "phone number" span two unrelated lines of prose
in a resume or README — exactly the kind of text this scrubber runs over. The
narrower class buys the reported format without that risk.

I also considered adding a separate alternation branch for the parenthesized
format specifically, leaving the existing branch untouched. I rejected it
because the evidence shows the gap is the separator class rather than the
parenthesized shape, so a parenthesized-only branch would fix the reported case
while leaving the same gap open for every other space-separated format.

Order of work: pattern first, confirm the behavior changes, then remove the
markers. That way the markers come off tests that are already observed to pass,
rather than on faith.

## Test plan

Re-run the commands from my reproduction comment against the change.

1. The issue's own snippet, which is the reproduction's primary artifact.
   Before (posted): the dashed number is redacted, the parenthesized one is
   not, and `detect()` returns a record only for the dashed one. Expected
   after: both numbers redacted, and `detect()` returning a `phone_us` record
   for the parenthesized number as well. The dashed-only control line must be
   unchanged from before.

2. The suite, via `python -m pytest tests/unit/test_pii_scrubber.py -v`.
   Before (posted): `20 passed, 5 xfailed`. Expected after: `24 passed,
   1 xfailed` — the four phone tests passing normally with their markers gone,
   and `test_mixed_pii_and_text` still XFAIL for the `street_address` reason
   above.

3. The pattern in isolation, to confirm no format regressed, against
   `555-123-4567`, `(555) 123-4567`, `(555)123-4567`, `555.123.4567`, and two
   false-positive probes: `I worked at TechCorp for 5 years` and
   `version 1.2.3 build 4567`. Expected after: the first four all match; the
   last two still return None. Widening a separator class makes a pattern
   hungrier, and these confirm it has not started matching version strings or
   ordinary prose containing numbers.

## Risks and unknowns

**The opening `(` is left unredacted.** With the change, the match on
`(555) 123-4567` starts at index 1 and spans `555) 123-4567`, so `scrub()`
returns `([REDACTED]` rather than `[REDACTED]`. No digits leak, but the paren
survives. This is a pre-existing property of the pattern: the word-boundary
assertion sits before the optional opening paren, and a word boundary cannot
match between the start of a string and `(`. It never surfaced before because
the space-separated case never matched at all. I am naming it as a known
limitation rather than fixing it, because moving that assertion is a separate
change to a separate part of the pattern and would widen this diff beyond the
one-line separator fix. The four tests I am unmarking assert on `[REDACTED]`
being present rather than on the exact output string, so this does not block
them — but it does mean the redaction is cosmetically imperfect, and a
maintainer may want it cleaned up separately.

**The `+1 555 123 4567` form now matches too.** It did not before, for the same
reason the parenthesized form did not: spaces. This is an intended side effect
rather than extra scope — it falls out of the same one-character change and
cannot be excluded without special-casing. `test_us_phone_formats` includes
space-separated formats, so this is consistent with what the tests already
expect.

**I am leaving one marker that cites this issue.** That is a judgment call
against a literal reading of CONTRIBUTING, and I may be wrong about it. My
reasoning is in Scope, with the evidence; if a maintainer reads the marker's
reason string as authoritative over the test's actual failure, I will remove it
and open a separate issue for the `street_address` over-match instead.

**Untested beyond this repo's suite.** I verified the new pattern against the
formats in `test_us_phone_formats` and the two false-positive probes above. I
have not run it against a large corpus of real prose, so I cannot rule out a
false positive on some text shape neither the suite nor my probes cover.

## Deviations

Nothing changed from the plan, and I checked rather than assumed. The build is
one commit, `e5ecc12`, with 1 insertion and 17 deletions across the two files
the plan named: the separator widening on line 16, and the four four-line
marker blocks.

All three of the test plan's predictions held:

1. The suite went from `20 passed, 5 xfailed` to `24 passed, 1 xfailed`, the
   remaining xfail being `test_mixed_pii_and_text` as expected.
2. `scrub()` now redacts both numbers in the combined string, and `detect()`
   returns two `phone_us` records where it previously returned one.
3. The dashed-only control is unchanged from the before run:
   `'Call me at [REDACTED]'`, one detection record at the same offsets.

The one thing the plan flagged as a known limitation also appeared exactly as
described: the output is `'Call me at ([REDACTED] or [REDACTED]'`, with the
opening paren surviving, and `detect()` reports the parenthesized number's
value as `'555) 123-4567'` rather than including the leading paren. That was
predicted under Risks, so it is not a deviation — but it is worth recording
that the prediction was confirmed rather than merely stated.

Because nothing changed, the plan comment I posted on the issue is still
accurate, so no correcting follow-up comment was needed.
