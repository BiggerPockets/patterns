---
categories: RSpec
name: Prefer match_array or contain_exactly over eq
---

When checking the contents of an iterable where the order doesnt matter, prefer RSpec's [`match_array`](https://www.rubydoc.info/gems/rspec-expectations/RSpec/Matchers#match_array-instance_method) or [`contain_exactly`](https://www.rubydoc.info/gems/rspec-expectations/RSpec/Matchers#contain_exactly-instance_method) over `eq`.

This practice help us avoid brittle test suites where a refactor may result in a different, but inconsequential, re-ordering of a function's results.

## Bad

In this example, using  `eq` makes an assumption about the behavior of the `User#interests` function as it relates to the order of `add_interest` function calls. This is brittle if `User#interests` is refactored to return this list in a different order.

```ruby
user.add_interest(interest_a)
user.add_interest(interest_b)
user.add_interest(interest_c)
expect(user.interests).to eq([interest_a, interest_b, interest_c])
```

## Good

Use `match_array` or `contain_exactly` when the order of of a function's results could be ambiguous.

```ruby
user.add_interest(interest_a)
user.add_interest(interest_b)
user.add_interest(interest_c)
expect(user.interests).to match_array([interest_a, interest_b, interest_c])
```

```ruby
user.add_interest(interest_a)
user.add_interest(interest_b)
user.add_interest(interest_c)
expect(user.interests).to contain_exactly(interest_a, interest_b, interest_c)
```
