---
categories: Practices
name: Code Review
---

Having Your Code Reviewed
-------------------------

* Write a PR title and description which explain the change being made to the system in the PR
* Include a link to the ticket/story from the pull request
* Be grateful for the reviewer's suggestions. ("Good call. I'll make that
  change.")
* Explain why the code exists. ("It's like that because of these reasons. Would
  it be more clear if I rename this class/file/method/variable?")
* Extract some tangential changes and refactorings into future tickets/stories.
* Push commits based on earlier rounds of feedback as isolated commits to the
  branch. Do not squash until the branch is ready to merge. Reviewers should be
  able to read individual updates based on their earlier feedback.
* Seek to understand the reviewer's perspective.
* Try to respond to every comment.
* Wait to merge the branch until continuous integration tells you the test suite is green in the branch.
* Request additional review after making major changes as a result of QA feedback
* Merge once you feel confident in the code and its impact on the system.

Reviewing Code
--------------

Understand why the change is necessary (fixes a bug, improves the user
experience, refactors the existing code). Then:

* Communicate which ideas you feel strongly about and those you don't.
* Identify ways to simplify the code while still solving the problem.
* Offer alternative implementations, but assume the author already considered
  them. ("What do you think about using a custom validator here?")
