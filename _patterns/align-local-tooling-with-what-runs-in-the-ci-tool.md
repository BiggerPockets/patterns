
---
categories: Practices
name: Align local tooling with what runs in the CI tool
---

## Problem: Local build not aligned with remote build


This is the "works on my machine" syndrome:

* I write some code, write tests for it
* I run linting tools in the command line or editor
* I run the tests and verify the behaviour locally
* I'm happy. I push up the code.

After waiting for the code to build, I'm told by the remote build tool that it's failed.

![image](https://user-images.githubusercontent.com/22185/151361649-84cbba49-6b4d-473a-90d5-9036ebda9d27.png)

I take a look. And here's what I see:

![image](https://user-images.githubusercontent.com/22185/151361717-f0296af3-b407-4fe0-8f7f-ffb7f5d67bec.png)

* That's **two lines** that are going to cost me the next **15 minutes** (`*`) at least.
* I fix those mistakes. I wait another 15 minutes.
* Whilst it's rebuilding I try to test it on the review app. But it's redeploying. So I have to wait.
* I test it on the review app. I find a mistake that's only exposed in review but I didn't see locally. Another 15 minutes.
* You get the idea. I've blown through my 4 hours budget very quickly, along with a good deal of my patience.

`*` based on a `building_remotely` time of 15 minutes as of 27/01/22.

## Solution: Local builds are exactly the same as remote builds

Example: linting.

We have a simple command line tool:

```bash
bin/lint
```

* Run locally whilst coding
* Remote build uses **this exact same tool**
* Same script to lint *all* languages - CSS, JS, Ruby
* Fast - runs in less than 20 seconds
* Autofixes mistakes where possible
* Option to run within editor to give live feedback

## Consequences

* Faster builds locally and remotely
* Confidence - if the script passes linting locally, you can be sure it'll pass remotely
* Less frustration for developers - no more waiting for 15 minutes only to find you've missed a freaking semicolon
* **Ultimately** - lower cycle time, faster development, better outcomes for everyone, higher performing team

## Related talks

[Elisabeth Hendrickson —  Care and Feeding of Feedback Cycles](https://www.youtube.com/watch?v=F0BRsnYwnBk)
