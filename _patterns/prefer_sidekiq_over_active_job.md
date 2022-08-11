---
categories: Rails
name: Prefer Sidekiq
---

When it comes to queueing systems, it sometimes feels like BiggerPockets has been around the world. Our current prefered underlying background job runner is Sidekiq. We also prefer to write jobs directly as Sidekiq workers rather than using the ActiveJob wrapper. Why?

- Sidekiq is faster than ActiveJob.
- Sidekiq jobs take up less RAM than ActiveJob, so we are less likely to end up in a scenario where we run out of space to queue more jobs.
- Only Sidekiq jobs can be enqueued using Sidekiq Enterprise cron jobs.
- Sidekiq has many more queueing options than whats capable using the ActiveJob wrapper.

## Bad

```ruby
class BackgroundJob < ActiveJob::Base
  def perform
    # do work
  end
end
```

## Good

```ruby
class BackgroundJob
  include Sidekiq::Worker

  def perform
    # do work
  end
end
```
