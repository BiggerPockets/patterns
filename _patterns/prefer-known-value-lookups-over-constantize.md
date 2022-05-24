---
categories: Rails
name: Prefer constant lookups over `constantize`
---

Rails `constantize` and `safe_constantize` methods are handy for writing dynamic code very quickly, but give maintainers no information about what should be allowed as a dependency. Data structures (sometimes known as registries) that take a known value and return a constant are easier to maintain and modify. This becomes especially important when class names have the possibility to be stored as strings in the database.

## Bad

This code assumes some form of `Step` will be passed to a class; however, it will accept any string.

```ruby
class Flow
  def current_step_from(step_name, params)
    step_name.constantize.new(params)
  end
end

# Flow.current_step_from("AboutYouStep", { foo: "bar" }) # => #<AboutYouStep { foo: "bar" }>
# Flow.current_step_from("Topic", { foo: "bar" }) # => instantiates an unexpected #<Topic { foo: "bar" }> object
```

## Good

A hash data structure helps maintainers know what types of objects should be returned from a dynamic function:

```ruby
class Flow
  STEPS = {
    "AboutYouStep" => AboutYouStep,
    "PickKeywordsStep" => PickKeywordsStep,
    "CompleteStep" => CompleteStep,
  }

  def current_step_from(step_name, params)
    STEPS.fetch(step_name).new(params)
  end
end

# Flow.current_step_from("AboutYouStep", { foo: "bar" }) # => #<AboutYouStep { foo: "bar" }>
# Flow.current_step_from("Topic", { foo: "bar" }) # => raises KeyError
```
