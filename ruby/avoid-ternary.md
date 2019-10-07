# Avoid using the ternary operator

Avoid using the ternary operator when writing a conditional. It is difficult to read and to write it, because having different operations on the same line can be confusing.

## Bad

````ruby
logged_in? ? "disabled" : ""
````

## Good

````ruby
if logged_in?
    "disabled"
else
    ""
end
````
