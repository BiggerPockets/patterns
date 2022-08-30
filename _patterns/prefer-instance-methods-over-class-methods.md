---
categories: Ruby
name: Prefer instance methods over class methods
---

* Class (aka static) methods [resist refactoring](http://blog.codeclimate.com/blog/2012/11/14/why-ruby-class-methods-resist-refactoring/)
* They're easier to directly reference in dependencies, which discourages refactoring and leads to tighter coupling
* They make additional dependency injection harder
* Class methods push bad dependencies up the stack

## Why do they make dependency injection harder?

* With class methods, you cannot store state in an instance. There is no instance.
* If you cannot store state, you cannot inject any dependencies in to the instance as state.
* You end up with a series of class methods, passing around dependencies in parameters
* This causes an increase in the number of arguments needed in each method
* This also means you can't pass instances of other objects into this object
* You end up with a series of namespaced functions with no state.
* Even if you want a set of pure functions for functional programming, instances are still more flexible - they are more compatible with dependency injection frameworks such as [`dry-auto-inject`](https://dry-rb.org/gems/dry-auto_inject/)

## Real Example - searching in Agent Finder

* We wanted to stop searching using `ElasticSearch` and instead use `Postgres`
* Use object oriented design - same interface for each, swap out different classes for different implementations

## Bad - using class methods

```ruby
# Search implementations
class PostgresSearch
  def self.call(query:)
    # ... do the search here ...
  end
end

class ElasticSearch
  def self.call(query:)
    # ... do the search here ...
  end
end

# Generic location search code
class LocationSearch
  def self.call(query:, search:)
    # ... snip ...
    search.call(query: query)
  end
end

# Controller
class SearchController < ApplicationController
  def show
    LocationSearch.
      call(query: params[:query], search: Rails.configuration.search)
  end
end

Rails.application.configure do
  # What if we need some config to be injected into these different implementations?
  # For example, host, username, password. Or maybe searching config such as exact match.
  # This would now be *impossible*.
  config.search = ElasticSearch # old
  config.search = PostgresSearch # new
end
```

## Good - using instance methods

```ruby
# Search implementations
class PostgresSearch
  def initialize(host:, username:) # Inject config, options or other instances
    @username = username
    @host = host
  end

  def call(query:)
    # ... do the search here ...
  end
end

class ElasticSearch
  def initialize(port: 1113)
    @port = port
  end

  def call(query:)
    # ... do the search here ...
  end
end

# Generic location search code
class LocationSearch
  def initialize(search:) # We can now dependency inject into the instance
    @search = search
  end

  def call(query:) # One less parameter then before
    # ... snip ...
    @search.call(query: query)
  end
end

# Controller
class SearchController < ApplicationController
  def show
    LocationSearch.
      new(search: Rails.configuration.search). # Inject configuration into instance
      call(query: params[:query])
  end
end

Rails.application.configure do
  # We can now inject options into the searchers at the top level. Yet more flexibility!
  config.search = ElasticSearch.new(port: 3445)                           # old
  config.search = PostgresSearch.new(host: "127.0.0.1", username: "root") # new
end
```
