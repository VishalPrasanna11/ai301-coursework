# Procedure: how this skill grades a PR package

## Read order

1. **Live mode only:** read `scope.md` first, before any other input. Confirm
   the `Repo:` line is filled and that the issue lives in that repo. If the line
   still carries a placeholder, stop without grading. Note any house rules it
   states.
2. Read the **issue** next: what behaviour is reported, and in the thread, any
   maintainer signal — a stated preference, a pointer, a rejected approach, a
   question. Write down the one sentence that says what goes wrong. In eval
   mode, this is the bundle's issue context.
3. Read the **repo's stated asks** third: the PR template's sections and the
   contributing policy's requirements, including any AI-use or disclosure rule
   and any named checks contributors must run. Note the exact wording of each
   requirement and the exact list of template sections. In eval mode, this is
   the bundle's repo-facts block.
4. Read the **plan** fourth, in full, including everything under `## Deviations`.
   Write down two lists: what the plan says is in scope, and what it says is out
   of scope. Treat a deviation note as an amendment of equal standing to the
   plan's body.
5. Read the **diff** fifth. List every file it touches and, for each, one
   sentence on what the change does. Do not judge yet.
6. Read the **test evidence** sixth, then the **PR title and description** last.
7. The order matters because five checks compare the package against something
   that must be fixed before the package is read. `diff-within-plan` and
   `plan-work-accounted` measure the diff against step 4's two lists;
   `evidence-observable` measures the evidence against step 2's behaviour and
   the plan's test plan; `template-sections-filled` and `policy-asks-met`
   measure the description against step 3's wording. Reading the description
   first would let its framing decide what counts as in scope, as tested, and as
   compliant — which is exactly the drift this order prevents.

## Evidence gathering

For each family, pull the fact from the location named and record it before
grading anything.

1. **Plan fidelity.** From step 5's file list and step 4's two lists, build a
   table: each changed hunk, and the plan line or deviation note that authorises
   it. Mark any hunk with no authorising line. Separately, mark any in-scope
   plan item with no corresponding hunk, and check the description for a
   sentence that accounts for it.
2. **Description against diff.** Take the description's Summary and Changes
   sections sentence by sentence. For each claim, find the hunk that supports
   it, or record that none does. Then take the diff's file list and, for each
   file, find the description line that accounts for it, or record that none
   does. Both directions matter: a claim with no hunk and a hunk with no claim
   are different failures.
3. **Test evidence.** Record, for each piece of evidence: the command shown, the
   output shown, and whether a before and an after both appear. Record which of
   the repo's named checks appear with real output, and for each, the result as
   stated. Where the evidence reports a failure, record the stated reason.
4. **Diff quality.** Walk the diff hunk by hunk. Record any commented-out code,
   debug or print statement, whitespace-only change, or committed working file.
   Separately, record any hunk that does not serve this issue's fix, and whether
   the fix forced it.
5. **Standards and comms.** Against step 3's list of template sections, record
   for each whether the description carries a real answer, a justified "N/A", or
   nothing. Against step 3's policy requirements, record for each the
   description text that meets it, or its absence. For a disclosure requirement,
   record the actual disclosing sentence; record "absent" if there is none,
   never "implied".
6. **Issue link.** Record the issue number the description names, and whether it
   matches the issue the plan belongs to.
7. If a family's evidence is genuinely absent, record it as absent rather than
   inferring it from a neighbouring part. An absence is a finding.

## Check execution

1. Execute the checks in this order: `diff-within-plan`, `plan-work-accounted`,
   `description-matches-diff`, `evidence-observable`, `repo-checks-run`,
   `no-debris`, `no-unrelated-hunks`, `template-sections-filled`,
   `policy-asks-met`, `issue-linked`, then the preferred checks
   `title-specific` and `commits-readable`.
2. Grade each check only against the evidence recorded in the gathering stage.
   Do not re-read the whole package for one check; if the record does not settle
   it, re-read only the part the evidence guide names for that family.
3. Grade `pass`, `fail`, or `unclear`, and record one piece of evidence: the
   hunk, the plan line, the command output, or the policy clause. For a `fail`,
   record the specific thing missing or contradicted, not a restatement of the
   check.
4. Where evidence for a check is genuinely absent, grade `unclear` rather than
   guessing a direction. Absent evidence is not a pass: a description that never
   mentions the repo's disclosure requirement has not met it, whatever the
   author intended.
5. Before failing any check on a shortfall — work not done, a check that failed,
   a limitation in the fix — look for a disclosure of that shortfall in the
   description or the deviations. If the package names it and gives the reason,
   the check passes. What fails is the gap between claim and content, not the
   shortfall.
6. Grade each check independently. Do not let an early fail short-circuit the
   rest: the value of the output is naming every problem at once, so the student
   fixes them in one pass rather than discovering them one run at a time.
7. Judge the outcome, never the shape. A description that answers a template
   section in one sentence of prose passes `template-sections-filled` on that
   sentence. The absence of a heading is not a finding; the absence of the answer
   is.

## Verdict assembly

1. Collect the grades for all `required` checks.
2. If every required check is `pass`, the verdict is `accept`. If any is `fail`
   or `unclear`, the verdict is `reject`.
3. `unclear` on a required check counts as a fail, per the rubric's verdict
   rule.
4. `preferred` grades never change the verdict. Record them, and use them only
   to say how far above the bar a passing package sits.
5. For each required check graded `fail` or `unclear`, quote in the summary the
   evidence that produced the grade: the hunk, the plan line, the missing
   output, or the unmet policy clause, as recorded during gathering. A student
   should be able to act on the summary without re-reading their own package.
6. In live mode, report voice-guide violations under a separate heading, and
   report any procedure gap you hit. Neither changes the verdict.
7. Emit the per-check grades with their evidence, then the fenced JSON block
   with the final verdict, and nothing after it. The same grades must always
   produce the same verdict; nothing in this stage is a judgment call.