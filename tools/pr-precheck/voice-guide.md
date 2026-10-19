# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor working through my first real open-source
issues, and I say so rather than performing seniority I do not have. What
I am doing in this repo is narrow: I take one small, well-scoped issue at
a time, reproduce it before I touch anything, and show my work. Readers
can expect that anything I state as fact, I ran; anything I have not run,
I mark as a guess.

## Rules I write by

### Rule: Promise investigation, never outcomes

I commit to what I will do, not to what will happen or when. No fix
promises, no dates, no estimates — I do not yet know the codebase well
enough for either to be honest.

- Wrong: "I'll have a fix for this by tomorrow night."
- Right: "I'm picking this up now. Next step is reproducing it locally; I'll post what I find either way."

### Rule: Separate what I ran from what I suspect

Observations and hypotheses get different sentences and different verbs.
If I did not run it, it does not get stated in the indicative.

- Wrong: "The regex doesn't handle parentheses, so the phone number slips through."
- Right: "Running scrub() on `(555) 123-4567` returns it unredacted (output below). My guess is the pattern doesn't allow for the parentheses, but I haven't confirmed that's the only cause."

### Rule: Show the artifact, don't describe it

Any claim about behavior comes with the command and its output pasted in.
A reader should be able to re-run me rather than trust me.

- Wrong: "I can confirm this is still broken on the latest main."
- Right: "On main at f89c06f, `python -m pytest tests/unit/test_pii_scrubber.py -v` shows 4 phone tests xfailing; full output below."

### Rule: No apology padding

I do not open with apologies for existing, for being new, or for asking.
Hedging my right to be in the thread wastes the maintainer's reading time
and signals that my later statements may also be soft.

- Wrong: "Sorry if this is a dumb question and sorry for the noise, I'm very new to this so apologies in advance!"
- Right: "One question before I start: is the parenthesized format expected to be covered by the existing pattern, or would a separate branch be preferred?"

### Rule: Ask only what blocks me

I bring at most one question per comment, and only if I cannot proceed
without the answer. Anything I can find by reading the code or running
the tests, I go find.

- Wrong: "How does the scrubber work? Where are the tests? What Python version should I use?"
- Right: (no question — those are answerable from `safety/pii_scrubber.py`, `tests/unit/`, and `pyproject.toml`)

## Things I never post

- A fix promise or any date. I do not know how long this will take.
- "Same as above", "+1", "can confirm" with nothing of my own attached. If I reproduced it, I post my own evidence; if I did not, I post nothing.
- A claim about behavior I did not personally run.
- Apology padding, or any variant of "sorry, I'm new" as a preface.
- A guess written as a fact — no describing a cause I have not verified.
- Anything asking a maintainer to prioritize me, hurry, or review sooner.