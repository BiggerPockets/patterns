---
categories: Rails
name: Be thoughtful about using UUIDs
---

For the most part, our data lives in a database (postgres) that has a robust and simple guaranteed uniqueness of identifiers.

A UUID is only useful when combining records that were committed to multiple, out-of-sync, sources. For example, you might choose to use UUIDs when you're developing a phone-based application that will periodically sync with a central service.

Carefully consider the costs of using a UUID versus the benefits. If there are no benefits, then use a traditional auto-incrementing database-backed ID column. You can always change to UUIDs later.
