---
categories: Ruby
name: Prefer Faraday as default HTTP client
---

Prefer Faraday as the default HTTP client

## Pros

* Global configuration options - open/read timeout, retries, etc.
* Configure retries using `FaradayMiddleware::Retry`
* Easily extend with middleware, e.g. [decoding JSON](https://github.com/BiggerPockets/biggerpockets/blob/37f31a80fe0f6a76c3b094fec706a1c7ac71e70d/app/models/insights/altos_rental_latest_date.rb#L32)
* Faster than httparty (speed comparison: https://www.scrapingdog.com/blog/ruby-http-clients/)
* Supports multiple adapters (Net::HTTP, Typhoeus, Patron, Excon, etc.)

## Bad

```ruby
require 'httparty'
require 'extended_httparty'

# GET request
response = HTTParty.get('https://jsonplaceholder.typicode.com/posts/1', timeout: 5)
# GET request with retry
response = ExtendedHttparty.get('https://jsonplaceholder.typicode.com/posts/1', tries: 5)
```

## Good

```ruby
require 'faraday'

# GET request
Faraday.get('https://jsonplaceholder.typicode.com/posts/1')

# GET request with retry
require 'faraday/retry'
conn = Faraday.new do |f|
  f.request :retry, {
    max: 5,
    interval: 0.05,
    interval_randomness: 0.5,
    backoff_factor: 2,
    timeout: 5,
  }
end

conn.get('https://jsonplaceholder.typicode.com/posts/1')
```
