---
categories: CSS
name: Consider SCSS variables
---

Often we need to write a CSS declaration of some value that is not found in one of the core variable files; particularly larger spacing dimensions. Instead of writing arbitrary looking dimensions, take the time to declare a well named variable in the CSS file. This will help a future reader understand the reasons why, and could lead to further reusability.

This is particularly useful when more than one dimension is needed for one specific element set (eg. a small, medium and large avatar)

### Bad
````html
<div class="avatar__badge-container">
  <%= image_tag(user.image_url) %>
</div>
````
````scss
.avatar__badge-container {
  height: 1.3rem;
  width: 1.3rem;
}
````

### Good
````html
<div class="avatar__badge-container">
  <%= image_tag(user.image_url) %>
</div>
````
````scss
$avatar-badge-container-small: 1.3rem;
$avatar-badge-container-large: 4rem;

.avatar__badge-container {
  height: $avatar-badge-container-small;
  width: $avatar-badge-container-small;
}
````
