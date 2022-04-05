---
categories: RSpec
name: Avoid randomness in specs
---

> In software testing, monkey testing is a technique where the user tests the application or system by providing random inputs and checking the behavior, or seeing whether the application or system will crash. Monkey testing is usually implemented as random, automated unit tests. - [Wikipedia](https://en.wikipedia.org/wiki/Monkey_testing)

Because we want to increasingly rely on automated tests for continuous deployment we're keen to avoid flaky (randomly failing) specs, so that our CI suite always passes.

These flaky specs are almost always caused by unpredictable inputs, for example random data generated with `.sample` or `Faker`

The randomness of monkey testing often makes bugs found difficult or impossible to reproduce. Unexpected bugs found by monkey testing can also be challenging and time consuming to analyze. In some systems, monkey testing can go on for a long time before finding a bug.

Try to avoid these in favour of deterministic spec inputs. Instead of relying on random data in tests, consider the edge/corner cases and write tests that explicitly exercise them. If test data should be unique consider using a FactoryBot [sequence](https://github.com/thoughtbot/factory_bot/blob/main/GETTING_STARTED.md#sequences).
