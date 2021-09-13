---
categories: CSS
name: Prefer Flat Stylesheets
---

Often, when writing CSS, we will choose to nest selectors, simply because it mirrors the hierachy of the companion markup. This has multiple disadvantages:

* Performance (it can easily spiral out of control and create very long selectors, resulting in a big asset file)
* Reusability (the styles defined in the innermost levels cannot be reused in other contexts)
* Confusing (cascade specificity rules in CSS are hard to keep track of)
* Pollution (by nesting selectors, we tend to keep names generic, which can cause conflicting styles -- see below)

Instead, we should aim to keep stylesheets as flat as possible, defining all selectors as top-level. This makes them globally **available for reuse**, and the flatness _enforces_ **specific naming** conventions in order to avoid style pollution.

When using SCSS, avoid the temptation to nest things using the parent selector (`&`) in order to reduce repetition. While this will still result in a flat stylesheet, it introduces its own problems: it is impossible to search for these selectors in your editor, and visually, it still appears as if the selectors are nested, which communicates the wrong thing.

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
  width: 20rem;

  .header {
    background: grey;

    .author-name {
      font-weight: bold;
    }
  }
}
````

### Bad
````scss
.blog-article {
  width: 20rem;

  &-header {
    background: grey;

    &-author-name {
      font-weight: bold;
    }
  }
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
  width: 20rem;
}

.blog-article-header {
  background: grey;
}

.blog-article-header-author-name {
  font-weight: bold;
}
````

## Exceptions

While flat stylesheets are preferable, there are some cases when nesting is actually preferred.

The most common exception is when the style rules defined would not make sense _without_ the rules defined in their parent. A good example of this is when you're working in a flexbox context:

````scss
.foo {
  display: flex;

  .bar {
    flex: 1 0 auto;
  }
}
````

Above, the `flex` rule defined within `.bar` would not do anything unless it was within the flexbox context established in its parent, `.foo`. In this case, reusability of the flexbox-dependent rules is _undesirable_, as using `.bar` anywhere except within `.foo` would result in unnecessary `flex` rule(s) being applied.

That is not to say that _all_ rules should be defined in a child selector, just because they are coupled with one rule that is. It is acceptable (even encouraged) to write multiple blocks that target the same selector - one as a child of another, and another that is at the top-level - which are responsible for different rules.

````scss
.foo {
  display: flex;

  .bar {
    flex: 1 0 auto;
  }
}

// this is the "true" definition of the component's style, while
// the above is how it should behave in just one specific context
.bar {
  color: green;
}
````

Above, the `color: green` rule is intended to be applied everywhere `.bar` is used, but the `flex` rule should only be applied when `.bar` is used in a specific context (a child of `.foo`).

This exception may also apply to styles that (unlike `flex` and `display: flex`) do not have any counterpart, and the reason for nesting can be more abstract. An example of this could be `margin`s.

Consider a mockup for a blog feature - a page that is a vertical list of multiple blog posts. Each blog post can be considered its own component. In the mockup, each blog post has 20 pixels of space between itself and the next, which will be represented in CSS as `margin-bottom`.

The definition of the reusable "blog post" component _should not_ include knowledge of this margin. This is another style rule that should only apply in a certain context: when the blog post is displayed alongside other blog posts. Because this rule defining spacing between the elements is dependent on context, nesting would be appropriate:

````scss
.all-blog-posts {
  .blog-post {
    margin-bottom: 20px;
  }
}

.blog-post {
  border: 1px solid red;
  padding: 10px;
}
````

A good rule of thumb (although this is not absolute) is to assume that rule sets in CSS should only include rules that apply _inside_ its box (the rectangle of space it consumes on the page). Margins, being "outside the box", should therefore not be considered rules defining the component, and instead _contextual_ rules.

In many cases, another example of this is `width`. The width of an element is often, deceptively, actually dependent on the width of its parent, even though a mockup may suggest a predefined width.

## Further Reading

* [Avoid nested selectors for more modular CSS](http://thesassway.com/intermediate/avoid-nested-selectors-for-more-modular-css)
