---
categories: Rails
name: Prefer Sidekiq
---

We prefer to write jobs directly as Sidekiq workers rather than using the ActiveJob wrapper. Why?

- Sidekiq is faster than ActiveJob.
- Sidekiq jobs take up less RAM than ActiveJob, so we are less likely to end up in a scenario where we run out of space to queue more jobs.
- Sidekiq Enterprise's cron feature requires Sidekiq workers and is incompatible with ActiveJob.
- Sidekiq has many more queueing options than what's capable using the ActiveJob wrapper.

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
