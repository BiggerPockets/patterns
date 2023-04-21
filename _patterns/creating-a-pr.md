---
categories: Practices
name: Pull Requests
---

## What are the goals of a PR?

Goals in order of importance:

1. Ensure correctness
2. Spread and share knowledge
3. Improve the design
4. Ensure consistency

## Guide for authors

### Who do I assign as a reviewer?

* Default to a single reviewer
* Reviewer should be selected based on:
  * Knowledge of reviewer (for example, if changing code related to events, @johngallagher has done work in this area)
  * Skills of reviewer (for example, @karlentwistle is skilled with performance testing)
* If other developers have more knowledge than the reviewer, the reviewer can pull them into the conversation
* For PRs in patterns, always assign the whole of Engineering

### Timescales

* The reviewer should give you a timescale when you can expect your PR to be reviewed.
* This means you can plan your day
* Look for a comment in the PR

### Scope creep

If you get feedback that's outside the scope of the ticket:
1. Create another ticket
2. Link to the ticket in a comment then resolve the comment


### What should be included in a PR?

PRs should be small and focussed.

Make sure you include everything on the [PR Checklist](#pr-checklist)

## Guide for reviewers

### What's the mindset?

1. To check the author's work for correctness
2. To prompt for better ways at approaching the problem
3. To look at the problem through fresh eyes
4. To encourage good design and progress of the codebase
5. To ask for the PR to be small enough so they can productively review it

### Timescale

* When asked to review a PR, please let the author know when you'll be able to review it
* Leave a comment on the PR tagging the author with the expectation, for example:

> @authorname I'll review this at 10am GMT Friday 13th March

### Scope creep

* If you give feedback that's outside the scope, bear in mind it won't necessarily be addressed within this PR
* The author should create a ticket for this follow up work, post a link and resolve the comment

### What should be included in a PR?

PRs should be small and focussed.

It's your right to ask for the PR to be broken down.

Check everything on the [PR Checklist](#pr-checklist)

## PR Checklist

### **Title**

1. Clear and detailed title

2. Title and body includes Jira ticket number e.g. `[BIG-6523]` or `[no-ticket]`

### **Description**

3. Contains link to Jira ticket

4. The **why** [will help us understand the change](https://www.pullrequest.com/blog/writing-a-great-pull-request-description/) 6 months from now

5. The **how** is detailed and includes links to the relevant source code

6. Manual test results included

### **Code**

7. Tested on a review app or locally

8. Adequate automated test coverage of changes made

9. Author has reviewed their own PR

10. Live review scheduled if change is non trivial

### **For UI changes**

11. PR includes before and after videos or screenshots

12. Tested on the [supported browsers](https://www.notion.so/Device-and-browser-testing-8bdb455a871c48b8acae1b6f1363c6eb), videos or screenshots included in the PR

### **For defects**

13. Regression test has been written that replicates the bug

### **For Segment tracking**

14. Tested events coming through via [the Segment debugger](https://app.segment.com/biggerpockets/sources/analytics_dev_environment/debugger)

15. Videos or screenshots taken of the events asserting what you expect - [see example](https://github.com/BiggerPockets/biggerpockets/pull/15150#issuecomment-1127803825)

### During Review

16. Author responses to reviewer comments are clear

17. Every reviewer comment has been responded to and resolved

18. When feedback has been actioned, the commit sha of the fix is in the comment and the comment has been resolved


## Example of a great PR

## [BIG-123] Add authentication support for dashboard

## What

I've added support for authentication to implement the dashboard. It includes model, table, controller and test.

## Why

These changes complete the user login and account creation experience. See #JIRA-123 for more information.

## How

This includes a migration, model and controller for user authentication. I'm using Devise to do the heavy lifting. I ran Devise migrations and those are included here.

## Testing

### 🟢 When user logs in they are shown the dashboard

✅ User is shown dashboard
✅ Logs indicate success

### 🟢 When a guest user visits the dashboard they are redirected to the home page

✅ User is shown home page
✅ Warning shown in logs

## Browser Testing

### Mobile

|  | OS | Device | OS version | Browser |
|--| --- | --- | --- | --- |
| 🟢 | iOS | iPhone 8 | 13  | Safari |
| 🟢 | iOS | iPad 5 | 11  | Safari |
| 🟢 | iOS | iPad Mini 4 | 11 | Safari |
| 🟢 | Android | Samsung Galaxy S21 | 11 | Samsung Internet |

### Desktop

|   | OS | Browser |
| -- | --- | --- |
| 🟢 | Windows 7 | Chrome 96 |
| 🟢 | Mac El Capitan (10.11) | Chrome 96 |
| 🟢 | Mac Sierra (10.12) | Safari 10.1 |
| 🟢 | Windows 10 | Edge 96 |
| 🟢 | Windows 7 | Firefox 59 |
| 🟢 | Mac El Capitan (10.11) | Firefox 78 |

## Screenshots

`<screenshots of the user logged in>`

## Anything Else?

Let's consider using a 3rd party authentication provider for this, to offload MFA and other considerations as they arise and as the privacy landscape evolves.

AWS Cognito is a good option, so is Firebase. I'm happy to start researching this path.

Let's also consider breaking this out into its own service. We can then re-use it or share the accounts with other apps in the future.
