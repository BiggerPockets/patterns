---
categories: RSpec
name: Write tests for public interfaces
---

> It is widely accepted that in most cases we should not be testing private methods, we should only test public methods to adequately describe object behaviour, let's make sure we add specs for all public interfaces.

We should strive to write specs for ALL public methods that are added to the codebase. This is particularly important when writing a new class, regardless of the simplicity of the method contents, as it is important to 'start as we mean to continue' with regards to documenting and promoting extensibility of classes.

This includes public instance methods, class methods, ActiveRecord scopes, controller actions (via integration tests).
