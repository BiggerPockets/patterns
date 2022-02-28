---
categories: Practices
name: Avoid subjective and time-based naming
---

Be as specific as possible with nameable concerns such as filenames, classnames, module names and method names. Avoid subjective and time-based terms such as 'new', 'improved', 'future' or '2022'. These types of non-specific term can rapidly lose meaning, and become difficult to remove from the codebase.

> Rails provides a lot of naming conventions we rely on frequently. The way things are named can have a lot of effect elsewhere so we should always try to be as specific as possible.

## Bad

```ruby
# app/views/webinars/big_marker/refactored_index.html.erb

<% if Flipper.enabled?(:big_marker_2022) %>
  <%= @improved_posts %>
<% end %>
```

## Good

```ruby
# app/views/webinars/big_marker/compact_index_view.html.erb

<% if Flipper.enabled?(:big_marker_compact_index_view) %>
  <%= @posts_without_user_attributes %>
<% end %>
```
