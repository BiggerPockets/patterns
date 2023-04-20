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

## Who do I assign as a reviewer?

* Default to a single reviewer
* Reviewer should be selected based on:
  * Knowledge of reviewer (for example, if changing code related to events, @johngallagher has done work in this area)
  * Skills of reviewer (for example, @karlentwistle is skilled with performance testing)
* If other developers have more knowledge than the reviewer, the reviewer can pull them into the conversation
* For PRs in patterns, always assign the whole of Engineering

## What's the mindset of reviewer?

1. To check the author's work for correctness
2. To prompt for better ways at approaching the problem
3. To look at the problem through fresh eyes
4. To encourage good design and progress of the codebase
5. To ask for the PR to be small enough so they can productively review it

## What should be included in a PR?

PRs should be small and focussed.

Reviewers - it's your right to ask for the PR to be broken down or that it includes any of the following checklist items.

Authors - before requesting reviews make sure your Pull Request adheres to the following checklist:

### **Title**

T1. Clear and detailed title

T2. Title and body includes Jira ticket number e.g. `[BIG-6523]` or `[no-ticket]`

### **Description**

D1. Contains link to Jira ticket

D2. The **how** is detailed and includes links to the relevant source code

D3. The **why** [will help us understand the change](https://www.pullrequest.com/blog/writing-a-great-pull-request-description/) 6 months from now

D4. Contains manual test results

### **Code**

C1. Tested on a review app or locally

C2. Adequate unit test coverage of changes made

C3. Author has reviewed their own PR

C4. In person review scheduled if change is non trivial

### **For UI changes**

U1. PR includes before and after screenshots

U2. Tested on the [supported browsers](https://www.notion.so/Device-and-browser-testing-8bdb455a871c48b8acae1b6f1363c6eb), screenshots included in the PR

### **For defects**

B1. Regression test has been written that replicates the bug

### **For Segment tracking**

TR1. Tested events coming through via [the Segment debugger](https://app.segment.com/biggerpockets/sources/analytics_dev_environment/debugger)

TR2. Screenshots taken of the events asserting what you expect - [see example](https://github.com/BiggerPockets/biggerpockets/pull/15150#issuecomment-1127803825)

### During Review

R1. Author responses to reviewer comments are clear

R2. Every reviewer comment has been responded to and resolved

R3. When feedback has been actioned, the commit sha of the fix is in the comment and the comment has been resolved

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
