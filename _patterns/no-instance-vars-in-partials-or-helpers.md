---
categories: Rails
name: Avoid using instance variables in view partials and helpers
---

It’s too easy to assume that certain instance variables are available in the context of your partial or helper. This leads to brittle and un-reusable partials and helpers that rely on mysterious other functions to set up their state for them. This is particularly bad for Rubyists since Ruby automatically casts undeclared instance variables to `nil`.

When we first extract a partial or helper, often for organisational reasons, it is typically used in only one place. When we reuse the partial somewhere else in our application, the new controller action in which it is called must correctly assign the same instance variable from the original controller action.

As our application becomes more complex, a view might contain several partials—each expecting their own instance variables—so we’ll need to assign several instance variables in the controller action. The context of these assignments will be a long way away from the nested partial in which they’re used.

If we rely on the “Rails magic” early on, we often pay in complexity later on. This is a pattern of many of the issues that people have with Rails’ HTML rendering environment. If we avoid using instance variables in our partial and helpers, they become simpler to reuse, even as our application grows in complexity.

## Bad

```erb
<%# app/views/album/show.html.erb %>

<% @album.songs.each do |song| %>
  <h2><%= song.name %></h2>
  <%= render "character", collection: song.characters,
                          as: :character %>
<% end %>
```

```erb
<%# app/views/album/_character.html.erb %>

<p><%= character.name %></p>
<div class="popup">
  Other Songs: <%= character.features_on_tracks(@album.songs).to_sentence %>
</div>
```

Not too obvious that the `_character` partial would need an `@album` instance variable defined, right?
Since all instance vars have `nil` value assigned by default, the exception might not be raised if we miss it

## Good

```erb
<%# app/views/album/show.html.erb %>

<% @album.songs.each do |song| %>
  <h2><%= song.name %></h2>
  <%= render "character", collection: song.characters,
                          as: :character,
                          locals: { songs: @album.songs } %>
<% end %>
```

```erb
<%# app/views/album/_character.html.erb %>

<p><%= character.name %></p>
<div class="popup">
  Other Songs: <%= character.features_on_tracks(songs).to_sentence %>
</div>
```

In this case an exception is always raised when the `songs` local variable is missed.

## Bad

```ruby
def show_free_ebook_ad?
  @forum.category == "books"
end
```

What happens if the `@forum` instance variable is renamed? It's also unclear what type of object the `@forum` instance variable refers to. This means that this method will be difficult to reuse and fragile to refactor.

## Good

Instead, pass dependencies into the helper method directly:

```ruby
def show_free_ebook_ad?(forum:)
  forum.category == "books"
end
```

Alternatively, rely on another helper method. A helper method here is preferable to an instance variable because, unlike instance variables, helper methods do not automatically cast to `nil`:

```ruby
def show_free_ebook_ad?
  current_forum.category == "books"
end
```
