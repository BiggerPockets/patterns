---
categories: Practices
name: Code Review
---

Having Your Code Reviewed
-------------------------

* Be grateful for the reviewer's suggestions.
* Feel free to disagree with the reviewer. Seek to understand the reviewer's perspective and strive for consensus.
* Keep your changes as small as possible. Leave tangential changes and refactorings for future commits.
* Request additional review after making major changes.
* Sometimes a bug or feature is best developed iteratively with successive deployments rather than exhaustive testing.

Reviewing Code
--------------

Understand why the change is necessary (fixes a bug, improves the user
experience, refactors the existing code). Then:

* Quickly and comprehensively review the pull request. Code that's not in production is not providing any value.
* Communicate which ideas you feel strongly about and those you don't.
* Focus on substantive issues rather than stylistic preferences. It's OK to
  comment on how you might do something differently and still approve the PR.
  Just because someone does it a different way doesn't mean that the code is
  wrong.
* Identify ways to simplify the code while still solving the problem.
* Offer alternative implementations, but assume the author already considered
  them. ("What do you think about using a custom validator here?")
* Be flexible, we don't exist in a vacuum. Software development requires a lot of tradeoffs and, occasionally, new tech-debt to meet an organization goal.
* Don't shave any yaks in a code review. That's why we have the patterns meeting.
