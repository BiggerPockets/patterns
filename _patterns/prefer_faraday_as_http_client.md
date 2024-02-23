---
categories: Ruby
name: Prefer Faraday as default HTTP client
---

Prefer Faraday as the default HTTP client because of the following:
1. It is faster
2. It is more flexible as it supports many underlying adapters
3. It does not make sense to have multiple HTTP clients in the same project

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

