---
categories: Ruby
name: Prefer instance methods over class methods
---

Class (aka static) methods [resist refactoring](http://blog.codeclimate.com/blog/2012/11/14/why-ruby-class-methods-resist-refactoring/). They're easier to directly reference in depending code, which discourages refactoring and leads to tighter code coupling.

## Bad

```ruby
class ServiceObject
  def self.perform(args)
    # ...
  end
end
```

## Good

```ruby
class ServiceObject
  def perform(args)
    # ...
  end
end
```
