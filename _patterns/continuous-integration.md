
---
categories: Practices
name: Continuous Integration
---

# Background

## What is continuous integration?

> Everyone commits to mainline every day.
>
> Jez Humble https://youtu.be/Y4H8dW7Ium8?t=1493

* We are *not* doing CI just because we use a CI tool (although that helps)
* Deploying once every day is an absolute minimum - ideally we should be doing it every few hours
* In order for us to achieve this, this we must **continously monitor and improve our feedback cycles**.

**Our Goal**: merging work into mainline every 4 hours.

## The four metrics to measure for high performing teams

From [Accelerate](https://www.amazon.co.uk/Accelerate-Software-Performing-Technology-Organizations-ebook/dp/B07B9F83WM/ref=sr_1_1?crid=RKBXDBF6G55H&keywords=the+science+of+devops&qid=1643288497&sprefix=the+science+of+devops%2Caps%2C76&sr=8-1):

1. **Cycle Time** - Time to implement, test, and deliver code for a feature (measured from first commit to deployment)
2. **Deployment Frequency** - Number of deployments in a given duration of time
3. **Change Failure Rate**  - Percentage of deployments which caused a failure in production
4. **Mean Time to Recovery** - Mean time it takes to restore service after production failure

# State of continuous integration - 27/01/22

## 1. Cycle time calculation

```
coding_locally + building_locally + testing_locally + building_remotely + deploying_review_app + testing_review_app + reviewing_pr + building_on_master + deploying_production
```

That's **nine** metrics which added up make the cycle time.

**Target cycle time** - 240 minutes (4 hours). Assuming all nine metrics take an equal length of time, that's just **26 minutes** for each stage.

And it gets a lot worse with other [antipatterns](#antipatterns) in play.

## Current metrics

`building_remotely` - 15 minutes
`deploying_review_app` - 5 minutes
`building_on_master` - 15 minutes
`deploying_production` - 5 minutes

# Antipatterns

1. Flaky tests. Penalty - 15 minutes per rebuild per branch.
2. Local build not aligned with remote build. Penalty - 20 minutes per uncaught mistake.
3. Mistakes made locally only caught in review app. Penalty - 20+ minutes per mistake.
4. Large PRs. Longer in coding. Penalty - more mistakes (every mistake made incurs > 15 minutes). More review cycles needed.
5. Slow PR review times. PRs are blocked waiting for other team members. Penalty - between 10 minutes and 1 day.
6. Review apps or production not aligned with local env. Penalty - between 15 minutes and a whole cycle time to fix.
7. Long remote build and deploy times. The longer these are, the more it hurts everything else.
