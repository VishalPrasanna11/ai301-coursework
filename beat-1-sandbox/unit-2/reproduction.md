# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

---

## Your identity upstream

**GitHub username**

VishalPrasanna11

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53#issuecomment-5864618494

Picking this up as well — several classmates have claimed it, so I'm not assuming exclusivity, just posting my own work. First contribution to this repo.

Claiming the investigation, not a fix or a date.

Next: reproducing the two reported behaviors — scrub() leaving (555) 123-4567 unredacted while redacting 555-123-4567 in the same string, and detect() returning [] for the parenthesized format — plus the four named tests in tests/unit/test_pii_scrubber.py, against current main. I'll post a report with my environment, commands, and output either way.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/53#issuecomment-5865057790

## Reproduction report

Reproduced as described, on macOS.

### Environment

| | |
|---|---|
| Commit | `f89c06f` (main, clean tree) |
| OS | macOS 14.4.1 (arm64) |
| Python | 3.12.4 |
| pytest | 9.1.1 |
| Install | `pip install -e ".[dev]"` in a venv |

Setup deviation: I did not run the documented `make setup` path (Docker, Postgres, Redis, `alembic upgrade head`, frontend install). `safety/pii_scrubber.py` is pure regex with no DB or API dependency, so I created a venv and installed the dev extra directly, then ran only this module's unit tests. Flagging it in case it matters to anyone re-running this.

### Steps

    $ git clone <fork of codepath/pathreview-ai301-fa26-s1>
    $ cd pathreview-ai301-fa26-s1
    $ python3 -m venv .venv
    $ source .venv/bin/activate
    $ pip install -e ".[dev]"
    $ python -m pytest tests/unit/test_pii_scrubber.py -v

### Observed: the issue's snippet, plus a control

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

Matches the issue: in the same string the dashed number is redacted and the parenthesized one is not, and `detect()` reports only the dashed one. The control shows phone redaction works in general, so the failure is specific to the parenthesized format rather than to the feature.

### Observed: the named tests

The four tests the issue names carry `xfail(strict=True)` markers referencing #53, so a plain run reports them as XFAIL rather than as failures:

    $ python -m pytest tests/unit/test_pii_scrubber.py -v
    ...
    test_us_phone_number_redaction XFAIL
    test_us_phone_formats          XFAIL
    test_detect_phone_pii          XFAIL
    test_phone_at_start_of_text    XFAIL
    test_mixed_pii_and_text        XFAIL
    ======================== 20 passed, 5 xfailed in 0.65s =========================

Re-running with `--runxfail` surfaces the underlying assertions:

    $ python -m pytest tests/unit/test_pii_scrubber.py -v --runxfail
    ...
    E       AssertionError: assert '[REDACTED]' in '(555) 123-4567 is my phone number.'
    tests/unit/test_pii_scrubber.py:200: AssertionError
    ========================= 5 failed, 20 passed in 0.14s =========================

### Mechanism, tested in isolation

`PIIScrubber.PII_PATTERNS["phone_us"]` is:

    \b(?:\+?1[-.]?)?\(?([0-9]{3})\)?[-.]?([0-9]{3})[-.]?([0-9]{4})\b

Each separator position accepts an optional dash or dot but not a space. Testing the pattern directly:

    $ python -c "
    import re
    from safety.pii_scrubber import PIIScrubber
    p = PIIScrubber.PII_PATTERNS['phone_us']
    for t in ['555-123-4567', '(555) 123-4567', '(555)123-4567', '555.123.4567']:
        print(repr(t), '->', re.search(p, t))
    "

    '555-123-4567'   -> <re.Match object; span=(0, 12), match='555-123-4567'>
    '(555) 123-4567' -> None
    '(555)123-4567'  -> <re.Match object; span=(1, 13), match='555)123-4567'>
    '555.123.4567'   -> <re.Match object; span=(0, 12), match='555.123.4567'>

Closing the space in the parenthesized form restores the match, which isolates the space itself — not the parentheses — as what breaks the case. So the gap is that no separator position admits a space.

### One test appears mismarked (not this issue)

`test_mixed_pii_and_text` carries an xfail marker citing #53, but its failing assertion is not about phone numbers. Under `--runxfail`:

    E       assert 'Python' in "\n        Professional Background:\n        I worked at TechCorp for [REDACTED]ications.\n        Email: [REDACTED]\n        Phone: [REDACTED]\n        SSN: [REDACTED]\n..."

The input was `I worked at TechCorp for 5 years developing Python applications.` Something redacted the span `5 years developing Python appl` — over-redaction of ordinary prose, the opposite defect from this issue. The phone in that test is `555-123-4567`, unparenthesized, and redacted correctly. I have not diagnosed which pattern causes it and am not treating it as evidence for #53; flagging it as a separate defect.

### Expected vs actual

**Expected:** `scrub()` redacts `(555) 123-4567` the same way it redacts `555-123-4567`, and `detect()` reports it.

**Actual:** reproduced as described — the parenthesized format passes through `scrub()` unredacted and `detect()` finds nothing for it, while the dashed format is handled correctly, on `f89c06f` in the environment above.

## Eval iterations

**Run history**

1. `--limit 3` smoke run — **2/3** (pkg-03 rejected against a gold label of accept, failing `environment-recorded` and `steps-executable`)
2. `--only pkg-01,pkg-02,pkg-03` after loosening both of those checks, with pkg-02 as a canary — **3/3**
3. Confirming full run, `--save-run eval-run.txt` — **19/20, PASS** (categories `clear-accept 7/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`)

Run 3 is the committed `eval-run.txt`.

**Package analysis**

**`pkg-03`** (BurntSushi/ripgrep#2779, adjacent replaced multiline matches producing wrong line numbers). My rubric: **reject** on the smoke run. Gold label: **accept**.

It failed two required checks, and both failures were my pass conditions rather than the package. `environment-recorded` as first written demanded "the commit or version of the code under test, the language runtime version, and the OS." The report's record is:

> Environment: ripgrep 15.2.0 (cargo install), Arch Linux (x86_64). The issue was filed against 13.0.0; behavior is unchanged on 15.2.0.

That names a release, an install method, an OS, and the version drift against the issue's target — everything a reader needs. But ripgrep is a compiled CLI tool with no language runtime a reader would separately install, so my "language runtime version" clause had no referent, and an unsatisfiable clause plus `unclear`-counts-as-fail meant an automatic reject.

`steps-executable` demanded that "every command needed to get from a clean checkout to the observed output is present and in order," and explicitly failed a step that was "described rather than given." The report says it created `test.txt` with the exact 12 lines from the issue, then gives the `rg` command that produces the output. Creating a fixture file is not a command, and the file's contents are already in the issue body verbatim, so a stranger can follow this exactly. My check was measuring the shape of the write-up rather than whether a reader could re-run it — the failure mode the rubric template warns about directly.

**Check rationale**

`steps-executable`, quoted as it reads now in the `rubric.md` uploaded to `tools/repro-check/`:

> A reader can get from nothing to the shown output by following the report. Commands that produce the output are given as runnable text. Setup that is not a command — creating a fixture file, editing a config — may be described, provided the description is precise enough to perform without guessing, including by pointing at exact content already present in the issue. Fails when a needed step is missing, when a description leaves a reader guessing at content or values, or when the sequence depends on state it never establishes

The first version read "Every command needed to get from a clean checkout to the observed output is present and in order, with no step referred to but not shown," and failed a step that was "described rather than given."

Two things were wrong with that. "From a clean checkout" assumes the software under test is a repository you build, which is true for the PathReview issue I am working on and false for a released binary like ripgrep — pkg-03's reader installs via cargo and never checks anything out. And "described rather than given" treats prose as automatically insufficient, when some setup genuinely has no command form: creating a fixture file, editing a config, clicking through a UI. The rubric template's own warning is that structure-shaped checks are what make graders disagree with themselves, and "must be a command" is a shape.

What replaced it is a test about the reader rather than about the text: can someone get from nothing to the shown output by following this? Prose setup passes when it is precise enough to perform without guessing, and the clause about pointing at content already in the issue is there because that is exactly what pkg-03 does. The three named failure modes — missing step, guessable values, unestablished state — are the cases where following the report actually breaks down.

**Trade-offs**

What this check now gives up is the ability to fail a report for vagueness that stops short of leaving a reader guessing. "Precise enough to perform without guessing" is a judgment call, and a report that says "set up a config with rate limiting enabled" would probably pass it while still costing a re-runner twenty minutes of figuring out which knob. The earlier version would have failed that outright. I accepted the looser test because the stricter one demonstrably failed a good report, and `unclear` counting as fail means every ambiguity in a required check becomes a rejection — that asymmetry is why all of my misses across both units have been wrongly rejected accepts rather than wrongly accepted rejects.

I re-ran with pkg-02 as a canary, since it was the `wrong-target` package already agreeing at reject and a loosened steps check could plausibly have let it through. It held at reject, and the confirming full run came back `unfollowable-comms 3/3`, so the category the change most directly touches did not degrade. The one remaining miss, pkg-05, fails on `steps-executable` and `control-shown` — so the loosening did not go far enough for that package rather than too far, which is the direction I would rather be wrong in.
