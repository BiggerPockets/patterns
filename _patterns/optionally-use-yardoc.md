---
categories: Ruby, Optional
name: Use YARD to document method signatures
---

[YARD](https://yardoc.org) is a Ruby documentation tool that's been around for a while. When used well, it can help
developers reason about the expectations that the code makes and know when it needs to be extended for new scenarios.

# What is YARD?

YARD is primarily a documentation tool. It's in use by many other teams and powers the
[RubyDoc.info](https://rubydoc.info) documentation generator. Most importantly for this topic, YARD helps maintain
developer sanity by giving us tools to document all the things that we expect a method to take, and what we expect it
to return or raise.

# A YARD example

Here's an example of using YARD in practice to document method signatures. This example highlights two YARD
keywords: `@param` and `@return`, but there are
[many others](https://rubydoc.info/gems/yard/file/docs/GettingStarted.md). The keywords are followed by an optional list
of everything that this method should be able to take, and what callers can expect to see returned.

```ruby
class Chatbot::CreateReplyToConversation
  # Append a reply to this conversation from the Chatbot, or do nothing if that fails.
  #
  # @param conversation [Converastion] the Conversation record between the user(s) and the chatbot.
  # @return [Message, nil] the response from the chatbot, or +nil+ if one was not generated.
  def perform(conversation)
    return unless conversation.present? # Make sure we're not dealing with a Null object if someone is being clever.

    create_reply_to(conversation)
  end

  # ...snip...
end
```

# Why

Ruby is a duck-typed programming language. The interpreter provides no guarantees that the code you write will actually
execute in production. Thus, developers need to have a very firm understanding of all of the ways that their code could
be called and what it might return.

Some teams have decided that the way to accomplish this is to use stricter type-safety, either provided
[by Ruby itself](https://github.com/ruby/rbs), or via 3rd party gems (such as [Sorbet](https://sorbet.org)). There is
even an entire language ([Crystal](https://crystal-lang.org)) that looks like Ruby, but dumps the interpreter for
compiled, statically-typed code.

At BiggerPockets, we have experienced the same problems that plauged these other teams: rampant calls to methods on
`nil`, missing constants in production, undefined variables, etc. We've looked into the alternatives that other teams
have chosen and, while good for those other teams, doesn't really fit well into our own preferred blend of Ruby. The
alternative we chose is one that gives the most teams the most flexibility to continue working on the monolith the same
way that they're always used to: [YARD](https://yardoc.org).
