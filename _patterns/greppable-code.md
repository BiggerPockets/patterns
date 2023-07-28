---
categories: Ruby, JavaScript, PHP, CSS, Rails
name: Write Greppable Code
---

When writing code, strive to ensure that _named things_ can quickly be found by their name. Avoid using interpolation, clever pattern matching, or string assembly to translate names because this makes them harder to find. Be explicit, don't invent names based on their context.

# Why?

Other teams in the business know only the name of the thing thing, not the implementation. Therefor, using their name means that the implementation is self-documenting, and other engineers can quickly use that name to find where it is used.

# Examples

## Ruby

In this example, assume that there are events generated for a "Sign Up Created" event. A non-engineering team uses this exact term to inform business decisions.

### Do this

```ruby
def track(event:)
  Analytics.track(event:)
end

track(event: "Sign Up Created")
```

### Not this

```ruby
def track(event:)
  event_name = event.split(".").last(2).map(&:capitalize).join(" ") # => "Sign Up Created"
  Analytics.track(event: event_name)
end

track(event: "membership.sign_up.created")
```

## Javascript

Don't use a Regular Expression when a plain ole string will be easier to grep for:

### Do this

```js
const ids = ["nav-desktop-menu-toggle", "nav-mobile-menu-toggle"];

if (ids.indexOf(someElement.id)) {
  // ...
}
```

### Not this

```js
const regex = /#nav-(desktop|mobile)-menu-toggle/;

if (regex.match(someElement.id)) {
  // ...
}
```

## HTML class attributes

Newer versions of CSS allow us to style things based on partial matches of the class name. This should be avoided.

### Do this

```css
div.alert-warning,
div.alert-info,
div.alert-success {
  /* styling */
}
```

### Not this

```css
div[class^="alert-"], div[class*=" alert-"] {
  /* styling */
}
```

# Reference

https://jamie-wong.com/2013/07/12/grep-test/
