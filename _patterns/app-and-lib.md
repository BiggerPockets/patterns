---
categories: Rails
name: App vs Lib
---

Ah Rails... so many directories! Where should I put my code?

So that we are consistent we choose to adopt the following:

## /app

- Our models, controllers, views, jobs, services, queries and mailers.
- Any other entity we create (class, module) that holds some **business logic**.

## /lib

- The /lib folder, as the name implies it, is meant to contain libraries. We know that in Ruby, libraries are packaged as gems most of the time. So this directory is for libraries that are not packaged as gems but could be.
- Put simply, ask yourself, "Could this code be a gem?". An example of code that might pass that test could be an API client.

## Further reading

https://devblast.com/b/rails-app-vs-lib
