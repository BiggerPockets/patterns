---
categories: Rails
name: Prefer SimpleDelegator over method_missing for wrapper classes
---

If you find yourself needing to make a light wrapper for an object to extend it with new methods, prefer using the standard
library's `SimpleDelegator` over implementing `method_missing` to reduce boilerplate.

Note that this pattern only applies to plain Ruby wrappers.

## Bad

```ruby
class Wrapper
  attr_reader :wrapped

  def initialize(wrapped)
    @wrapped = wrapped
  end

  def expired?
    wrapped.fetch(:expired_at) > Time.now
  end

  def method_missing(meth, *args, **kwargs, &block)
    if wrapped.respond_to?(meth)
      wrapped.send(meth, *args, **kwargs, &block)
    else
      raise NotImplemented
    end
  end

  def respond_to_missing?(meth)
    wrapped.respond_to?(meth)
  end
end
```

## Good

```ruby
class Wrapper < SimpleDelegator
  attr_reader :wrapped

  def initialize(wrapped)
    super(wrapped)
    @wrapped = wrapped
  end

  def expired?
    wrapped.fetch(:expired_at) > Time.now
  end
end
```
