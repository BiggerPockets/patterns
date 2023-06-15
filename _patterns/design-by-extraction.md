---
categories: Our Practices
name: Design by Extraction
---

If you're presented with a problem that has already been solved by loads of duplication and you want to reduce some of
that duplication by introducing a design, make sure that whatever design you're proposing reduces some of that
duplication, and make a plan to replace the other duplications with your new design.

It's better to be intentional about global rewrites than to solve one problem one time with a one-off new design.

# Why?

Code should be exemplary. If you add a new design but only put it in one place, how will other engineers know
to use it? The new design has not reduced overall complexity.

It should be obvious that there is now _a better way_ because alternative ways have been removed.

Consider this scenario:

1. Alice is working on a problem and realizes that it's a variation of problems that she's solved before. For example,
   Alice may be building a calculator that requires calculating APR-per-month, and she notices that there are many
   copies of functions that calculate APR per month.
1. Alice decides that she's going to write a new global APR-per-month function and use that in her new calculator
   report. She indends for this new global function to be reused elsewhere, so she adds it to a module and then
   inlcudes that module in her report, but in doing so, **she only replaces the function in her report.**
1. Bob notices Alice's new module and how it could enable consolidation of many similar APR-per-month functions.
   During code review, Bob makes a note to Alice that it would be beneficial to spend time refactoring the other
   instances of the similar functions so that total duplication is reduced, especially since Alice has introduced a
   new global function to accomplish the calculation.

Of course, there are caveats to this:

- It's generally a bad idea to refactor whilst introducting new functionality. Alice should consider "pre-factoring"
  the new generic APR-per-month calculation before adding her new report.
- Sometimes, duplication is necessary, and we should err on the side of more duplication than less. Less duplication
  can be brittle to decouple from special scenarios.
