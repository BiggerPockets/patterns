---
categories: Ruby, Optional
name: Use YARD to document method signatures
---

[YARD](https://yardoc.org) is a Ruby documentation tool that's been around for a while. When used well, it can help
developers reason about the expectations that the code makes and know when it needs to be extended for new scenarios.

# An Example

Here's an example of using YARD in practice to document method signatures. This example highlights two YARD
keywords: `@param` and `@return`, but there are
[many others](https://rubydoc.info/gems/yard/file/docs/GettingStarted.md).

```ruby
# @param market [Market]
# @param current_user [User, nil]
# @return [Array<Agent>, nil]
def perform(market:, current_user: nil)
  # ...
end
```

The benefit of the annotation is reduced ambiguity. The `current_user` param can probably be inferred from its name, but
the `market` param cannot: is a String or other object expected? An developer would need to read the method and
potentially investgate other callers to see how this method is used.

# Why

Ruby is a duck-typed programming language. The interpreter provides no guarantees that the code you write will actually
execute in production. Thus, developers need to have a very firm understanding of all of the ways that their code could
be called and what it might return.

Some teams have decided that the way to accomplish this is to use stricter type-safety, either provided
[by Ruby itself](https://github.com/ruby/rbs), or via 3rd party gems (such as [Sorbet](https://sorbet.org)). There is
even an entire language ([Crystal](https://crystal-lang.org)) that looks like Ruby, but dumps the interpreter for
compiled, statically-typed code.
