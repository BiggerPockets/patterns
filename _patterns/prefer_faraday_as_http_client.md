---
categories: Ruby
name: Prefer Faraday as default HTTP client
---

Prefer Faraday as the default HTTP client because of the following:
1. It is faster than Httpparty (Speed comparison: https://www.scrapingdog.com/blog/ruby-http-clients/)
2. It has a retry ability with the `FaradayMiddleware::Retry` middleware
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

### Retry ability

Httpparty does not have any mechanism for retrying requests.

There is a gem called extended_httpparty (https://github.com/zaagan/extended-httparty)
that adds retry ability. But this is not very popular and does not seem to be maintained.
Also, retries have be set on each request.

```
response = ExtendedHttparty.get('http://api.stackexchange.com/2.2/questions?site=stackoverflow', tries: 5)
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

### Retry ability

Faraday has a middleware called `FaradayMiddleware::Retry` that can be used to retry requests.
This is more flexible than extended_httpparty the Faraday instance can be initialized
globally with retry options.

```ruby
require 'faraday'
require 'faraday/retry'

retry_options = {
  max: 2,
  interval: 0.05,
  interval_randomness: 0.5,
  backoff_factor: 2
}

conn = Faraday.new(...) do |f|
  f.request :retry, retry_options
  #...
end
```
