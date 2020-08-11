---
title: A few thoughts on automated testing
date: "2019-06-05T22:10:12Z"
---

In a past job interview, a company asked me to write a response to the following question: 

*What are some of your opinions on JavaScript testing? If you were brought in as a consultant for a company that had no tests, how would you sell them on JavaScript testing and what is some advice you’d offer the engineers writing the tests?*

Since then I've reflecting back on my answers, made a few edits, and decided they would make for a good first blog post. 

---

For a developer:
- Write tests in a way that reflects how the program or functionality is going to be consumed and validates behavior in a way you can safely change the underlying implementation.

- A well tested codebase has a combination of low and level tests. Low-level tests (e.g. unit tests) validate small pieces of functionality, and high-level tests (integration, e2e, API endpoint tests) validate large chunks of functionality that comprise smaller pieces.   

- A high-level test:
    - validates behavior that is more closely tied to tangible business value.
    - implicitly validates the layers of implementation details underlying that behavior
    - can be written in a way that self-documents the publicly consumable behavior and helps developers understand it

- A low-level/unit test:
    - helps isolate root-cause of failure when high-level tests fail
    - runs fast because it tests a small piece of the overall behavior 
    - is useful for increasing test coverage because it can isolate fine-grained execution paths
    - can be colocated/organized according to its corresponding functionality, which helps maintainability

- TDD/test-first works well for pure functions their because behavior can be distilled down to expected outputs given inputs. Examples: data transformers/munging, utility functions, React/Redux reducers and selectors.

- Tests reduce the risk of breakage during refactoring. During initial development, write tests against the public facing API. Then during refactoring, make incremental change while using those tests to validate the behavior of the new code.

- Remember to test against any 3rd party API's and external dependencies which you consume but are out of your control, so that you will know if their changes cause you breakages.

- Ideally strive for 100% code coverage, but in practice be wary of diminishing returns past a certain coverage percentage.

For company/management:
- With each change to your software, you have to fully test whether the change 1) works as intended and 2) did not break other behavior. Otherwise, you risk introducing bugs that impact business performance.

- With manual testing, the marginal cost of fully testing your software increases exponentially with the size of the software. With automated testing, the marginal cost of fully testing remains directly correlated with the size of the respective feature.
