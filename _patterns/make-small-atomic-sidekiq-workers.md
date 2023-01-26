---
categories: Rails
name: Make small, atomic Sidekiq workers
---

We had a [serious issue](https://www.notion.so/biggerpockets/Lead-bundle-subscriptions-incorrectly-pausing-or-resuming-298e9b328c834b148e3c79f12a686004) where user subscriptions were not being updated.

* Break down Sidekiq workers into the smallest functional unit you can
* If one database failure can block other objects from being updated, that's a sign your design is wrong
* Each Sidekiq job should be able to error independently of others
* Use either batch jobs in Sidekiq enterprise or have one job enqueue others

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
    Rails.error.handle(ActiveRecord::RecordNotFound) do
      lead_bundle_subscription = LeadBundleSubscription.find(lead_bundle_subscription_id)
      if lead_bundle_subscription.pause_end_date.past?
        lead_bundle_subscription.unpause!
      else
        lead_bundle_subscription.pause!
      end
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