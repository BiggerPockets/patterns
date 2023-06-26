---
categories: Ruby
name: Prefer the Colorado Comma
---

When authoring Hashes, Arrays, or multi-line named function arguments, add the _Colorado Comma_, a comma that appears
after the last item in the collection.

# Bad

```ruby
hash = {
  foo: :bar,
  baz: :qux
}

array = [
  :foo,
  :bar
]

method_call(
  foo: :bar,
  baz: :qux
)
```

# Good

```ruby
hash = {
  foo: :bar,
  baz: :qux,
}

array = [
  :foo,
  :bar,
]

method_call(
  foo: :bar,
  baz: :qux,
)
```

# Why?

It makes subsequent diffs cleaner. Consider these two diffs for adding a new item to a hash:

Without the Colorado Comma:

```diff
  hash = {
-   foo: :bar
+   foo: :bar,
+   baz: :qux,
  }
```

With the Colorado Comma:

```diff
  hash = {
    foo: :bar,
+   baz: :quz,
  }
```

The diff with the pre-existing Colorado Comma is shorter.
