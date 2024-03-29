---
categories: Ruby, Optional
name: Use YARD to document method signatures
---

[YARD](https://yardoc.org) is a Ruby documentation tool that's been around for a while. When used well, it can help
developers reason about the expectations that the code makes and know when it needs to be extended for new scenarios.

The primary benefit of YARD annotations is reduced ambiguity.

# Example

## Bad

Consider this method:

```ruby
def perform(market:, current_user: nil)
  # ...
end
```

The `current_user` parameter can probably be inferred from its name, but the `market` parameter cannot: is a `String` or
some other object expected? A developer would need to read the method and potentially investgate other callers to see
how this method is used.

## Good

Here's the same method with YARD annotations:

```ruby
# @param market [Market]
# @param current_user [User, nil]
# @return [Array<Agent>, nil]
def perform(market:, current_user: nil)
  # ...
end
```

Its's now obvious what the method expects for the the `market` parameter. It
also lets the developer know that the method returns an `Array` composed of `Agent` objects, but it may
also return `nil`, so callers need to be ready to handle that.

# Why

Ruby is a duck-typed programming language. The interpreter provides no guarantees that the code you write will actually
execute in production. Thus, developers need to have a very firm understanding of all of the ways that their code could
be called and what it might return.

Some teams have decided that the way to accomplish this is to use stricter type-safety, either provided
[by Ruby itself](https://github.com/ruby/rbs), or via 3rd party gems (such as [Sorbet](https://sorbet.org)). There is
even an entire language ([Crystal](https://crystal-lang.org)) that looks like Ruby, but dumps the interpreter for
compiled, statically-typed code.
