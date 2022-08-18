---
categories: Rails
name: Set a TTL on every cached object
---

Caching is hard enough on its own, and it's only made harder when trying to debug a user hitting a stale cache. Instead of letting a cached object become very stale, set a sane TTL on it so that it automatically expires.

## Good

```ruby
Rails.cache.fetch("some-key", expires_in: 12.hours) do
  an_expensive_operation
end
```

## Bad

```ruby
Rails.cache.fetch("some-key") do
  an_expensive_operation
end
```
