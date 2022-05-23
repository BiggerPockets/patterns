---
categories: CSS
name: Mostly use REM units for spacing, sizing and media queries
---

To provide the best UX and accessibility on the site we should use REM units for spacing. REM unit spacing is calculated by the browser based on the root font size. We work from designs with specific pixel measurements in mind so it makes sense we take a flexible approach to spacing (font size, margin, padding, widths) amounts where possible.

Pixel units are fixed and do not scale with browser settings.

EM units may sometimes be suitable, but they can be much harder to reason about. The actual rendered size is based on the nearest parent with a font size, so the final calculation may be relative to complicated nesting logic. Some developers prefer to use EM for media queries, but this becomes tricky when a max and min width is declared.

## Exceptions

It is usually necessary to use fixed Px dimensions for images/image containers so the images inside scale correctly.
It is necessary to use fixed Px dimensions for 'line-height' so copy horizontal spacing scales correctly.

> Take a second to see if the REM sizing you are using exists already as a variable, and if it doesn't it probably should, to maintain consistent styling on the site. Take a look in app/assets/stylesheets/bp2017/. Particularly: _spacing.scss, _typography.scss, _variables.scss _breakpoints.scss and _colors.scss

### Bad
````html
<article class="blog-article">
  <div class="header">
    Written by <span class="author-name">Biggie Pockets</span>
  </div>
</article>
````
````scss
.blog-article {
  padding: 32px;
}
````

### Bad
````scss
.author-name {
 . font-size: 14px;
}
````

### Good
````html
<article class="blog-article">
  <div class="blog-article-header">
    Written by <span class="blog-article-author-name">Biggie Pockets</span>
  </div>
</article>
````
````scss
.blog-article {
  width: 2rem;
}

.blog-article-author-name {
  font-size: 0.825rem;
}
````

## Further Reading

* [REM vs Px vs EM](https://engageinteractive.co.uk/blog/em-vs-rem-vs-px)
