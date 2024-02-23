---
categories: Ruby
name: Prefer Faraday as default HTTP client
---

Prefer Faraday as the default HTTP client because of the following:
1. It is faster than Httpparty (Speed comparison: https://www.scrapingdog.com/blog/ruby-http-clients/)
2. It has a retry ability at the middleware layer
3. It is more flexible as it supports many underlying adapters
4. It does not make sense to have multiple HTTP clients in the same project. When we have
   one we can set one open/read timeout and other global configuration options.

## Bad

```
require 'httparty'

# Make a GET request
response = HTTParty.get('https://jsonplaceholder.typicode.com/posts/1')

# Print response information
puts "Response code: #{response.code}"
puts "Response body: #{response.body}"
```

## Good

```
require 'faraday'

# Create a Faraday connection
conn = Faraday.new(url: 'https://jsonplaceholder.typicode.com')

# Make a GET request
response = conn.get('/posts/1')

# Print response information
puts "Response code: #{response.status}"
puts "Response body: #{response.body}"
```
