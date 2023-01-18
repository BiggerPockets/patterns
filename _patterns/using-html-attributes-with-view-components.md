---
categories: Rails
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

  def initialize(variant:, html: {})
    @variant = variant
    @html = html
  end
  
  def css_classes
    "#{html[:class]}"
  end
  
  def html_attributes
    html.except(:class)
  end
end
```

```erb
<!-- the template -->
<%= content_tag(:div, class: css_classes, **html_attributes) do %>
  <%= content %>
<% end %>

<!-- using the component -->
<% render(ViewComponent.new variant: "number", html: { id: "foo", class: "blue", data: { fizz: :buzz }}) do %>
  <!-- content -->
<% end %>
```
