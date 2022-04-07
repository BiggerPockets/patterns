---
categories: Rails
name: Hyphenated URL paths
---

We prefer paths with hyphens rather than underscores.

URLs where words are broken with hyphens are more easily read and remembered, because hyphens are recognisable English punctuation. Hyphenated URLs are also the agreed-upon convention for most of the Internet, so sticking to hyphens eliminates the chance of mixing underscores and hyphens in URLs.

The potential SEO benefits are rumoured, but unconfirmed. That said, [Google recommends hyphens](https://support.google.com/webmasters/answer/76329?hl=en) over underscores, and Google does at least [distinguish between the two](https://www.mattcutts.com/blog/dashes-vs-underscores/).

## Bad

```rb
    resources :renewal_discounts
```

## Good

```rb
    resources :renewal_discounts, path: "renewal-discounts"
```
