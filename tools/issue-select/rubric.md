cat > ~/.claude/skills/issue-select/rubric.md << 'EOF'
# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| repo-alive | repo-facts block: date of the most recent commit to the default branch, compared to the snapshot date | The most recent default-branch commit is within 90 days of the snapshot date | required |
| maintainer-present | Comment thread: comments by anyone marked maintainer, owner, member, or collaborator. repo-facts: recent maintainer activity | A maintainer has commented in this issue's thread, or repo-facts shows maintainer activity within 30 days of the snapshot date | required |
| scope-bounded | Issue body: the change being requested | The body names a specific file, function, message, or behavior to change, and the work reads as a localized edit. Requests to design, refactor, migrate, or add a subsystem fail | required |
| behavior-specified | Issue body. For a defect: the steps, command, or input that produce it, and the expected result. For a docs or feature request: the desired end state | The body contains a reproduction or a concrete end state. A symptom with no way to trigger it fails, and a request with no stated target state fails | required |
| no-open-design-question | Comment thread, reading from the last message backward | No maintainer question about whether or how to do this is left unanswered | required |
| unclaimed | Assignee field; claim statements in the comment thread; any linked open pull request | No assignee, no unretracted claim, and no open pull request linked to the issue | required |
| newcomer-labeled | Issue labels | Carries good first issue, beginner, help wanted, or an equivalent label | preferred |
| test-surface | Issue body and repo-facts: whether the affected area has existing tests | The change has a nearby test file, or the issue states a way to verify the fix | preferred |

## Verdict rule

Accept if and only if every `required` check passes. A single required
fail rejects the issue.

`unclear` counts as fail on a required check: an issue whose suitability
cannot be verified from the evidence available is not an issue a newcomer
should take.

`preferred` checks never change the verdict. They rank the accepted
issues against one another, with `newcomer-labeled` weighted above
`test-surface`.
EOF
cat ~/.claude/skills/issue-select/rubric.md