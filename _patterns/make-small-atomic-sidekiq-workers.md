---
categories: Rails
name: Make small, atomic Sidekiq jobs
---

## Problem

When Sidekiq jobs are big with too much functionality:

* Can cause defects as we're lumping together behaviour that doesn't necessarily belong together
* Can significantly hold up other jobs, blocking the queue and potentially breaching the queue SLA (e.g. `within_five_minutes`)

## Solution

* Break down Sidekiq jobs into the smallest functional unit you can
* Each Sidekiq job should be able to error independently of others
* For dependent jobs, either batch jobs in Sidekiq enterprise or have one job enqueue others

## Bad

```ruby
class StartStopPauseJob
  include Sidekiq::Job

  def perform
    LeadBundleSubscription.find_each do |lead_bundle_subscription|
      # ⚠️ This will make a huge database query, only to discard 90% of the records!
      next unless lead_bundle_subscription.pause_start_date

      # ⚠️ If any of these lines cause a crash, it aborts the whole job and valid subscriptions aren't updated
      if lead_bundle_subscription.pause_end_date.past?
        lead_bundle_subscription.unpause!
      else
        lead_bundle_subscription.pause!
      end
    end
  end
end
```

## Better

```ruby
class StartStopPauseJob
  include Sidekiq::Job

  def perform
    LeadBundleSubscription.with_pause_start_date.find_each do |lead_bundle_subscription|
      StartStopPauseSubscriptionJob.perform_async(lead_bundle_subscription.id)
    end
  end
end

class StartStopPauseSubscriptionJob
  include Sidekiq::Job

  def perform(lead_bundle_subscription_id)
    lead_bundle_subscription = LeadBundleSubscription.find(lead_bundle_subscription_id)
    if lead_bundle_subscription.pause_end_date.past?
      lead_bundle_subscription.unpause!
    else
      lead_bundle_subscription.pause!
    end
  end
end
```

## Best

```ruby
class ActivateOrPauseJob
  include Sidekiq::Job

  def perform
    # Use #in_batches and #perform_bulk for performant queries and minimise Redis round trips
    LeadBundleSubscription.with_pause_start_date.in_the_past.in_batches do |subscriptions_batch|
      ActivateJob.perform_bulk(subscriptions_batch.ids.zip)
    end

    LeadBundleSubscription.with_pause_start_date.in_the_future.in_batches do |subscriptions_batch|
      PauseJob.perform_bulk(subscriptions_batch.ids.zip)
    end
  end
end

# Small atomic jobs
class PauseJob
  include Sidekiq::Job

  def perform(lead_bundle_subscription_id)
    LeadBundleSubscription.find(lead_bundle_subscription_id).pause!
  end
end

class ActivateJob
  include Sidekiq::Job

  def perform(lead_bundle_subscription_id)
    LeadBundleSubscription.find(lead_bundle_subscription_id).activate!
  end
end
```