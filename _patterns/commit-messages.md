---
categories: Practices
name: Better commit messages and clean git history
---

## A good commit message

### First line
The first line should be 50 or less characters and serve as a summary. Capitalize it and omit any trailing punctuation.

### Further description
If necessary add further description after the first line text and a blank line. Also add a link ticket/story if relevant.

Tim Pope describes a good commit message in the post [A Note About Git Commit Messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

```
Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.  Wrap it to about 72
characters or so.  In some contexts, the first line is treated as the
subject of an email and the rest of the text as the body.  The blank
line separating the summary from the body is critical (unless you omit
the body entirely); tools like rebase can get confused if you run the
two together.

Write your commit message in the imperative: "Fix bug" and not "Fixed bug"
or "Fixes bug."  This convention matches up with commit messages generated
by commands like git merge and git revert.

Further paragraphs come after blank lines.

- Bullet points are okay, too

- Typically a hyphen or asterisk is used for the bullet, followed by a
  single space, with blank lines in between, but conventions vary here

- Use a hanging indent
```


### Message

Write meaningful messages. Messages like: "Set env variable X" are not very useful as that can be seen from looking at the change itself.
A message like "Set variable to enable Y service on Heroku" might be better.

A commit message should aim to answer the questions like:
- Why was this change made?
- How does it solve the issue?
- What are the possible side effects?

If you start by writing a good summary of the changes in the PR you can copy that into the squashed commit before merging.

#### Read more here
Tips on how to write a better commit messages
https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message


## Squashing commits before merge to master

### Why
Squashing commits before merging to master makes debugging easier and keeps the commit history clean.
When you are looking at why some changes were made broken commits on the master make it difficult to find why the change has been implemented. You need to search for the relevant PR and if even that doesn't have a good description it just makes it hard or impossible to find the reason why someting was done.

PRs often include many commits reflecting developer's work, with added small commits that fix linter errors, typos and adressing the PR feedback. We end up with many commits that mess up commit history.
That is why it is suggested that once the PR gets approved the commits should be squashed into one with a meaningful commit message before merging.
Sometimes it still makes sense to keep multiple commits before merging, especially when they would help future developers as distinct commits in the git history.

### How

When merging a PR in Github you can change the button `Merge pull request` by clicking on the right side of it and selecting `Squash and merge`. This will also give you an option to change the message of the squashed commit.
