---
categories: Rails
name: Build views with view components
---

When working on the view layer in the application (app/views) prefer using view
components over the alternatives:

* Partials
* ERB files with business logic

### Flexible components

Make sure to generalize your use case so that the component can be used in any
part of the application.

In order to make components more reusable the library provides us with concepts like
the `content` helper and [slots](https://viewcomponent.org/guide/slots.html).

Always prefer extending component or composing new ones with existing ones over 
creating additional components. Feature specific components are welcomed, but they 
should be composed by other reusable components. There are more details on this topic 
[here](https://viewcomponent.org/viewcomponents-at-github.html#the-two-types-of-viewcomponents-we-write).

Here's an example that help illustrate how components can be flexible:

#### Bad
```rb
<%= render BigButton.new(text: "Big button") %>
<%= render ButtonWithIcon.new(text: "Button name", icon: "assets/icons/star.svg") %>
```

#### Good
```rb
<%= render Button.new(text: "Big button", size: :large) %>
<%= render Button.new(text: "Button name") do |component| %>
  <% component.with_leading_icon(icon: "assets/icons/star.svg") %>
<% end %>
```

### Tested components

Make sure that view components are tested. The ViewComponent library introduces a
new type of test (`type: :component`) which pulls in a few useful helpers that
make testing components quite easy.

Familiarize yourself with [view component testing](https://viewcomponent.org/guide/testing.html).

#### Why is it important to test view components?

- It reduces the number of necessary system tests.
- View component tests are much faster than system tests.
- These tests allow us to easily check for edge cases.

Here's an example that help illustrate how simple a component test can be:

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

### Styled components

Avoid using writting CSS when building view components. We're moving away
from using custom CSS files and using [Tailwind](https://tailwindcss.com) 
instead. If you're having a hard time styling a component using Tailwind reach 
out to someone from the design team to discuss making adjustments to the 
component.

Here's an example that help illustrate how using Tailwind looks as opposed
to writting custom CSS:

#### Bad
```css
# app/components/button.scss
.button {
  &-content { ... }
  &-label { ... }
}
```
```rb
# app/components/button.html.erb
<button type="button" class="button">
  <span class="button-content">
    <span class="button-label">Button text</span>
  </span>
</button>
```

#### Good
```rb
# app/components/button.html.erb
<button type="button" class="inline-flex items-center rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm">
  Button text
</button>
```

### Shared components

Always check [our custom component library](https://www.biggerpockets.com/lookbook) 
before creating a new one. The goal of our component library is not to grow 
indefinitely but instead to provide the building blocks to implement the view layer 
of all user facing features.

To make sure new components are included in the component library
a [component preview](https://viewcomponent.org/guide/previews.html) is required. 
Previews should include [annotations](https://lookbook.build/guide/previews/annotating/) 
and parameter descriptions. It should also ideally present the default use of 
the component along with common variations. Component previews automatically get 
included in the Lookbook.

Here's an example of what a component preview looks like:
```rb
# spec/components/previews/button_component_preview.rb
class ButtonComponentPreview < ViewComponent::Preview
   # Playground
   # ---------------
   # This is the base button that can be customized as described by the
   # parameters.
   #
   # @param type select { choices: [button, submit, reset] } "Defaults to `:button`"
   # @param size select { choices: [small, medium, large] } "Defaults to `:medium`"
   # @param scheme select { choices: [primary, secondary] } "Defaults to `:primary`"
   # @param expand_on_mobile select { choices: [true, false] } "Defaults to `false`"
   def playground(type: :button, size: :medium, scheme: :primary, expand_on_mobile: false)
     render ButtonComponent.new(type:, size:, scheme:, expand_on_mobile:) do
       "Button text"
     end
   end

   # Secondary
   # ---------------
   # This is the secondary scheme.
   def secondary
     render ButtonComponent.new(scheme: :secondary) do
       "Secondary button"
     end
   end
 end
```
