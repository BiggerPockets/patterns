---
categories: Ruby, JavaScript, PHP, CSS, Rails
name: Write Greppable Code
---

When writing code, strive to ensure that _named things_ can quickly be found by their name. Avoid using interpolation or string assembly to translate names because this makes them harder to find.

# Why?

Other entities in the business know only the name of the thing thing, not the implementation. Therefor, using their name means that the implementation is self-documenting, and other engineers can quickly use that name to find where it is used.

# Example

In this example, assume that there are events generated for a "Sign Up Created" event. Other organizations use this exact event name to make business decisions.

## Do this

```ruby
def track(event:)
  Analytics.track(event:)
end

track(event: "Sign Up Created")
```

## Not this

```ruby
def track(event:)
  event_name = event.split(".").last(2).map(&:capitalize).join(" ")
  Analytics.track(event: event_name)
end

track(event: "membership.sign_up.created")
```

# Reference

https://jamie-wong.com/2013/07/12/grep-test/
