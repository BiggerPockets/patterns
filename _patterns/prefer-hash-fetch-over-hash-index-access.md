---
categories: Ruby
name: Prefer Hash#fetch over Hash#[]
---

[`Hash#fetch`](https://ruby-doc.org/core-3.1.1/Hash.html#method-i-fetch) is prefered over [`Hash#[]`](https://ruby-doc.org/core-3.1.1/Hash.html#method-c-5B-5D) because it makes you think about how you want to handle the possibility of the key not being defined. This helps avoid a bunch of subsequent `nil` checks, especially if you're getting data from an external source. If you're trying to be careful about introducing `nil`s, using `fetch` can be really helpful.

## Bad

```ruby
hash = { a: "1" }
hash[:b] # => nil
```

## Good

An `KeyError` exception can help you write less brittle code because it's harder to introduce the possibility of `nil`.

```ruby
hash = { a: "1" }
hash.fetch(:b) # raises KeyError
```

It's even better when you know what a good default value should be when a key is not defined:

```ruby
hash = { a: "1" }
hash.fetch(:b, "go fish") # => "go fish"
```
