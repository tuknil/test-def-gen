# Repository Instructions

## Pull Request Review Policy

1. Perform one comprehensive initial review covering P0-P2 findings.
2. Record findings with stable IDs; do not repeatedly restart an unbounded full review.
3. P0 and P1 findings are merge blockers.
4. P2 and lower findings are non-blocking:
   - Document them in Jira or the pull request.
   - Do not request another review cycle solely for P2 or lower findings.
5. Remediation reviews must verify:
   - Previously reported findings.
   - Regressions introduced by the remediation.
   - P0 or P1 issues in newly changed code.
6. A newly discovered P0 or P1 during remediation remains merge-blocking even if it was missed in the initial review or is outside the newly changed lines. Record it with a stable ID and explain why it surfaced during verification.
7. New P2 or lower findings discovered during remediation are follow-up work, not merge blockers.
8. Recommend merge when:
   - No unresolved P0 or P1 findings remain.
   - Required tests and security gates pass.
   - The branch is current and mergeable.
9. Do not expand review scope after reaching the P2-only state unless material new code is added or a P0/P1 is discovered.
10. The normal maximum is one full review plus one focused verification review.