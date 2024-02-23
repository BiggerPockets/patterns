---
categories: Ruby
name: Prefer Faraday as default HTTP client
---

Prefer Faraday as the default HTTP client because of the following:
1. It is faster than Httpparty (Speed comparison: https://www.scrapingdog.com/blog/ruby-http-clients/)
2. It allows for a global connection configuration for setting open/read timeout, retries, etc.
3. It has advanced retry ability with the `FaradayMiddleware::Retry` middleware
4. It is more flexible as it supports many underlying adapters
5. It does not make sense to have multiple HTTP clients in the same project. When we have
   one we can set one open/read timeout and other global configuration options.

## Bad

Httpparty has to be configured at a request level so request options have to be explicitly
set for each request. This is not ideal because it is easy to forget to set
a configuration and it is harder to have unified configuration options for all requests.
Also, the retry gem associated with Httpparty is not very popular and does not have a lot
of timeout options.

```ruby
require 'httparty'

# Make a GET request
response = HTTParty.get('https://jsonplaceholder.typicode.com/posts/1', timeout: 5)

# Print response information
puts "Response code: #{response.code}"
puts "Response body: #{response.body}"
```

External gem for retrying requests with Httpparty

```ruby
response = ExtendedHttparty.get('http://api.stackexchange.com/2.2/questions?site=stackoverflow', tries: 5)
```


## Good

Faraday can be set up at a connection level with global configuration options. This means
that all requests made with the Faraday instance will have the same configuration options
such as open/read timeout, retries, etc.

Retries can be set up with the `FaradayMiddleware::Retry` middleware.


```ruby
require 'faraday'
require 'faraday/retry'

retry_options = {
  max: 2,
  interval: 0.05,
  interval_randomness: 0.5,
  backoff_factor: 2,
  timeout: 2,
}

conn = Faraday.new(...) do |f|
  f.request :retry, retry_options
  #...
end

con.get('https://jsonplaceholder.typicode.com/posts/1')
```
```
