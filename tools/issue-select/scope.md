cat > ~/.claude/skills/issue-select/scope.md << 'EOF'
# Scope: where to look, and who is looking

<!--
This file is the skill's field of view. The rubric (rubric.md) decides
whether an issue is GOOD; the scope decides which issues are candidates
at all, and whose hands the issue would land in. It applies in live mode
only: in eval mode the bundle is the whole world and this file is
ignored.

Two parts. Staff wrote the first; you write the second.
-->

## Where candidates come from

Only issues in the course's Path Review repository are candidates:

- Repo: `codepath/pathreview-ai301-fa26-s1`

Do not search, fetch, or grade issues from any other repository, however
promising. The wider GitHub comes later in the course; for now the field
is Path Review.

**Path Review house rule.** Path Review is a classroom, and your
classmates are not strangers. Ignore the usual claim signals here: other
students' claim comments (and there may be several on one issue) do not
block an issue, and finding some on the issue you want is normal. Claim
anyway: course credit attaches to the pull request you open, not to
whether it merges, so a shared issue costs nobody anything. Everything
else in the rubric applies as written.

## Your fit profile

I have written non-trivial code in Python and in JavaScript/TypeScript,
mostly application-level work: scripts, small web apps, and coursework
projects. I am comfortable reading unfamiliar code in either language and
running a project's test suite. I have not worked in compiled languages,
and I have not maintained build or CI configuration beyond following
setup instructions.

What I want out of a first issue is practice reading a real codebase well
enough to make a small, correct change in it.

Rank higher: issues in Python or JavaScript/TypeScript; issues where the
fix lives in application code rather than build tooling; issues where an
existing test can tell me whether I got it right.

Rank lower: issues requiring an unfamiliar toolchain or a heavy local
setup, since that time goes to the environment instead of the change.

I have no strong preference between frontend and backend work.
EOF
cat ~/.claude/skills/issue-select/scope.md