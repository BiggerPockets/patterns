---
categories: Rails
name: Scheduled jobs
---

When you have a job you want scheduled to run at specific intervals, much like cron in a traditional server environment, use [Sidekiq Enterprise periodic jobs](https://github.com/mperham/sidekiq/wiki/Ent-Periodic-Jobs).

## Bad

Do not use [Heroku Scheduler](https://devcenter.heroku.com/articles/scheduler)

## Good

app/jobs/background_job.rb
```ruby
class BackgroundJob
  include Sidekiq::Worker
  sidekiq_options queue: "cron"

  def perform
    # do work
  end
end
```

config/initializers/sidekiq.rb
```ruby
config.periodic do |mgr|
  mgr.register("*/5 * * * *", "BackgroundJob")
end
```
