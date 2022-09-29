---
categories: Rails
name: Use design first methodology and OpenAPI
---

When designing APIs, we can use a "design first" or "code first" methodology.

[This article explains the different approaches](https://apisyouwonthate.com/blog/api-design-first-vs-code-first).

At BiggerPockets we're choosing [design first, evolve with code](https://apisyouwonthate.com/blog/api-design-first-vs-code-first#design-first-evolve-with-code) because it's mature and gives us many advantages.

### Advantages

* Save masses of time building API endpoints
* Reduces support requests from partners
* Enforces correctness
* Improvements to cross cutting concerns (e.g. logging) are cheap
* Get mock servers for free
* Docs are always up to date
* Docs look professional, thanks to [Stoplight](https://stoplight.io/)
* Compatible with JSON schema for event validation

### Disadvantages

* Need to decouple the API from Rails
* More work up front
* Learning curve
