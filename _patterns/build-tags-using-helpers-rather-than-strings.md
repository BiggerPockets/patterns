---
categories: Rails
name: Use tag helpers rather than strings to assemble HTML
---

While slightly more verbose, Rails' tag helpers have several advantages over composing HTML directly via strings. Helpers automatically sanitize HTML, preventing code injection attacks called Cross-Site Scripting (XSS).

## Bad

Here's an example of vulnerable code:

```erb
<div class="<%= color %>"></div>
```

Why is this dangerous? If color can be set by a user, a hacker could inject javascript:

```ruby
color = "/><script>alert('hacked!')</script>"
#=> <div class="/><script>alert('hacked!')</script>"></div>
```

## Good

Use the content_tag helper, which properly sanitizes text content and attributes:

```ruby
color = "/><script>alert('hacked!')</script>"
content_tag(div, class: color)
#=> <div class="/&gt;&lt;script&gt;alert(&#39;hacked!&#39;)&lt;/script&gt;"></div>
```

## Further reading

https://cheatsheetseries.owasp.org/cheatsheets/Ruby_on_Rails_Cheat_Sheet.html
