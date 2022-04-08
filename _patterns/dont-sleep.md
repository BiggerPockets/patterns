---
categories: RSpec
name: Don't use `sleep` in specs
---

Sometimes if we're waiting for some javascript to run or an operation to complete we might add a `sleep` in our specs.

The problem is that this can create non-deterministic specs and forces our entire test suite to slow down based on how many seconds we `sleep`!

## Bad

```ruby
sleep 1
```

## Good

```ruby
# Capybara will automatically wait until element is visible, up until a timeout
expect(find("#element")).to have_content("login failed")
```

## Bad

```ruby
stub_const("Livestream::LiveEvent::EXPIRATION_TIME", 1.second)

Livestream::LiveEvent.update(foo: "bar")

expect do
  sleep 1.5
end.to change { Livestream::LiveEvent.exists? }.from(true).to(false)
```

## Good

```ruby
stub_const("Livestream::LiveEvent::EXPIRATION_TIME", 1.second)

Livestream::LiveEvent.update(foo: "bar")

expect do
  travel 2.seconds
end.to change { Livestream::LiveEvent.exists? }.from(true).to(false)
```

