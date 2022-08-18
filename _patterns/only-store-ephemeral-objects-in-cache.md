---
categories: Rails
name: Only store ephemeral objects in the cache
---

The application cache should never be used as the sole store for long-lived objects. An engineer should be able to run `Rails.cache.clear` at any time and not break a feature or raise any exceptions.

## Good

```ruby
def user_clicked_link(user, link)
  LinkClick.create!(user:, link:)
  Rails.cache.delete("clicked-link/#{user.id}/#{link.id}")
end

def user_clicked_link?(user, link)
  Rails.cache.fetch("clicked-linked/#{user.id}/#{link.id}", expires_in: 1.week) do
    LinkClick.exist?(user:, link:)
  end
end
```


## Bad

```ruby
def user_clicked_link(user, link)
  Rails.cache.write("clicked-link/#{user.id}/#{link.id}", true)
end

def user_clicked_link?(user, link)
  !!Rails.cache.read("clicked-link/#{user.id}/#{link.id}")
end
```
