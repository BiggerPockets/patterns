---
categories: Rails
name: Send Segment events inline rather than in background jobs
---

Segment's documentation recommends [not sending events from Sidekiq or other batch job systems](https://segment.com/docs/connections/sources/catalog/libraries/server/ruby/#getting-started)

> The analytics-ruby gem makes requests asynchronously, which can sometimes be suboptimal and difficult to debug if you're pairing it with a queuing system like Sidekiq/delayed job/sucker punch/resqueue. If you prefer to use a gem that makes requests synchronously, you can use simple_segment , an API-compatible drop-in replacement for the standard gem that does its work synchronously inline. If you choose to use simple_segment, please note that because the simple_segment package isn’t owned and maintained directly by Segment, Segment won't be able to provide support for it.

Their API client is [configured by default to collect events in batches of 100 in a background queue](https://segment.com/docs/connections/sources/catalog/libraries/server/ruby/#performance).

In addition:

* We enqueue roughly **3 million** tracking related jobs a month - a significant load on Redis
* This generates roughly **6 million** log entries a month - which costs us in Datadog

## Bad

````ruby
class TrackingWorker
  include Sidekiq::Job

  def perform(attributes)
    Analytics.track("Some event", attributes)
  end
end

class SomeController < ApplicationController
  def create
    # ... some work here ...
    TrackingWorker.perform_async
  end
end
````

## Good

````ruby
class SomeController < ApplicationController
  def create
    # ... some work here ...
    Analytics.track("Some event", attributes)
  end
end
````

