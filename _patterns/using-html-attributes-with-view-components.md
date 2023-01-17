---
categories: View Components
name: "Using HTML Attributes in View Components"
---

When authoring a view component, in general you want to be able for the result to be used in ways that are already supported by the browser. For that reason, each ViewComponent initializer method should take a generic keyword argument named "attributes" that is passed through to the view as the HTML attributes. Why? This keeps our ViewComponent implementations small and flexible.

# Bad

```ruby
class MyViewComponent < ApplicationViewComponent
  attr_reader :id, :class_name, :variant, :stuff_for_data

  def initialize(id:, class_name:, variant:, stuff_for_data:)
    @id = id
    @class_name = class_name
    @variant = variant
    @stuff_for_data = stuff_for_data
  end
end
```

```erb
<!-- the template -->
<%= content_tag(:div, id:, class: "some-class #{class_name}", data: stuff_for_data) do %>
  <%= content %>
<% end %>

<!-- using the component -->
<% render(ViewComponent.new id: "foo", class_name: "blue", variant: "number", data: { fizz: :buzz }) do %>
  <!-- content -->
<% end %>
```

# Good

```ruby
class MyViewComponent < ApplicationViewComponent
  attr_reader :variant, :attributes

  def initialize(variant:, **attributes)
    @variant = variant
    @attributes = attributes
  end
end
```

```erb
<!-- the template -->
<%= content_tag(:div, class: "some-class #{Array(attributes[:class]).join(" ")}", **attributes.except(:class)) do %>
  <%= content %>
<% end %>

<!-- using the component -->
<% render(ViewComponent.new id: "foo", class: "blue", variant: "number", data: { fizz: :buzz }) do %>
  <!-- content -->
<% end %>
```
