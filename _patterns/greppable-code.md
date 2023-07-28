---
categories: Ruby, JavaScript, PHP, CSS, Rails
name: Write Greppable Code
---

When writing code, strive to ensure that _named things_ can quickly be found by their name. Avoid using interpolation, clever pattern matching, or string assembly to translate names because this makes them harder to find.

# Why?

Other teams in the business know only the name of the thing thing, not the implementation. Therefor, using their name means that the implementation is self-documenting, and other engineers can quickly use that name to find where it is used.

# Example

In this example, assume that there are events generated for a "Sign Up Created" event. A non-engineering team uses this exact term to inform business decisions.

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

# HTML class attributes

Newer versions of CSS allow us to style things based on partial matches of the class name. This should be avoided.

# Do this

```css
div.alert-warning,
div.alert-info,
div.alert-success {
  /* styling */
}
```

# Not this

```css
div[class^="alert-"], div[class*=" alert-"] {
  /* styling */
}
```

# Reference

https://jamie-wong.com/2013/07/12/grep-test/
