---
categories: Practices
name: Pull Requests
---

What makes a good pull request?
-------------------------

* Write a clear, concise PR title *and* description which explain the change being made to the system in the PR
* Include a link to the ticket/story from the pull request
* If there's anything you're unsure of, or would like the reviewer to focus on please mention that in the description
* Include some simple guidelines on what QA might be needed
* Before marking the PR as ready for review, take a minute to do the first round of code review yourself. If there's anything obvious to you, address those issues
* If your work requires manual acceptance testing, do a cursory smoke test on your review app to ensure everything is in place (data, API keys, etc.) for QA Engineers to perform verification

Assigning reviewers
-------------------------

Pull Reminders will randomly assign a reviewer. Also consider requesting additional review. Factors that you might consider whilst selecting additional reviewers:

- The original author of the code being changed may have valuable insight
- A specific developer may have expertise in the subject of the PR (e.g. email system, search, Redis, Marketplace)
- Your PR may be interesting to others if it introduces a new concept or framework
