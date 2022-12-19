---
categories: Rails
name: Build views with view components
---

When working on the view layer in the application (app/views) prefer using view
components over the alternatives:

* Partials
* ERB files with business logic
* Layouts

### Reusability

Make sure to generalize your use case so that the component can be used in other
parts of the application.

In order to make components more reusable the library provides us with concepts like
the `content` helper and [slots](https://viewcomponent.org/guide/slots.html).

Feature specific components are welcomed, but they should consist of other reusable
components. There are more details on this topic [here](https://viewcomponent.org/viewcomponents-at-github.html#the-two-types-of-viewcomponents-we-write).

Here's an example that help illustrate reusability:

#### Bad
```rb
# app/views/actions.html.erb
<button type="button" class="Button--secondary Button--medium Button">
  <span class="Button-content">
    <span class="Button-label"><%= I18n.t("accept") %></span>
  </span>
</button>
<button type="button" class="Button--primary Button--medium Button">
  <span class="Button-content">
    <span class="Button-label"><%= I18n.t("cancel") %></span>
  </span>
</button>
```

#### Good
```rb
# app/views/actions.html.erb
<%= render(Button.new(text: :accept) %>
<%= render(Button.new(text: :cancel, scheme: :primary) %>
```

### Testing

Make sure that view components are tested. The ViewComponent library introduced a
new type of test (`type: :component`) which pulls in a few useful helpers that
make testing components quite easy.

Familiarize yourself with [view component testing](https://viewcomponent.org/guide/testing.html).

#### Why is it important to test view components?

- It reduces the number of necessary system tests.
- View component tests are faster than system tests.
- These tests allow us to easily check for edge cases.

Here's an example that help illustrate the benefits:

```rb
# app/components/table_component.rb
class TableComponent < ApplicationComponent
  def initialize(collection:)
    @collection = collection
  end

  def render?
    @collection.any?
  end
end

# spec/components/table_component_spec.rb
RSpec.describe TableComponent, type: :component do
  it "is not rendered when the collection is empty" do
    render_inline(TableComponent.new(collection: []))
    expect(page).not_to have_selector("table")
  end
end
```

### Component library

Always check for existing components before creating a new one. Our custom
component library lives [here](https://lookbook.build) to list all of the
existing components so that we avoid duplication.

Prefer extending existing components and making them more general over creating
specialized components that are unlikely to be reused.

Here's an example that help illustrate that:

### Bad
```rb
<%= render(ButtonWithIcon.new(text: "Button name", icon: "assets/icons/star.svg")) %>
```

### Good
```rb
<%= render(Button.new(text: "Button name")) do |component| %>
  <% component.with_leading_icon(icon: :star) %>
<% end %>
```

### Styling

Avoid using custom styles when building view components. We're moving away from
using custom CSS files and using [Tailwind](https://tailwindcss.com) instead.
If you're having a hard time styling a component using Tailwind reach out to
someone from the design team to see what adjustment can be made.

Here's an example that help illustrate reusability:

#### Bad
```rb
# app/views/button.html.erb
<button type="button" class="button--secondary button--medium button">
  <span class="button-content">
    <span class="button-label">Button text</span>
  </span>
</button>
```

#### Bad
```rb
# app/views/button.html.erb
<button type="button" class="inline-flex items-center rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm">
  Button text
</button>
```
