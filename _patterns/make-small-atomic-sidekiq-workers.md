---
categories: Rails
name: Make small, atomic Sidekiq workers
---

We had a [serious issue](https://www.notion.so/biggerpockets/Lead-bundle-subscriptions-incorrectly-pausing-or-resuming-298e9b328c834b148e3c79f12a686004) where users' subscriptions were not being updated.

* Break down Sidekiq workers into the smallest functional unit you can
* If one database failure can block other objects from being updated, that's a sign your design is wrong
* Each Sidekiq job should be able to error independently of others
* Use either batch jobs in Sidekiq enterprise or have one job enqueue others

## Bad

```ruby
module Companies
  module LeadBundleSubscriptions
    class StartStopPauseJob
      include Sidekiq::Worker

      def perform
        ::Companies::LeadBundleSubscription.find_each do |lead_bundle_subscription|
          next unless lead_bundle_subscription.pause_start_date # This will make a huge database query, only to discard 90% of the records!

          # If one of these crashes, it aborts the whole job and a bunch of valid subscriptions aren't updated
          if lead_bundle_subscription.pause_end_date.past?
            lead_bundle_subscription.unpause!
          else
            lead_bundle_subscription.pause!
          end
        end
      end
    end
  end
end
```

## Better

```ruby
module Companies
  module LeadBundleSubscriptions
    class StartStopPauseLeadBundleSubscriptionJob
      include Sidekiq::Worker

      def perform(lead_bundle_subscription_id)
        Rails.error.handle(ActiveRecord::RecordNotFound) do
          lead_bundle_subscription = ::Companies::LeadBundleSubscription.find(lead_bundle_subscription_id)
          if lead_bundle_subscription.pause_end_date.past?
            lead_bundle_subscription.unpause!
          else
            lead_bundle_subscription.pause!
          end
        end
      end
    end

    class StartStopPauseJob
      include Sidekiq::Worker

      def perform
        ::Companies::LeadBundleSubscription.with_pause_start_date.find_each do |lead_bundle_subscription|
          StartStopPauseLeadBundleSubscriptionJob.perform_async(lead_bundle_subscription.id)
        end
      end
    end
  end
end
```

## Best


```ruby
module Companies
  module LeadBundleSubscriptions
    class PauseJob
      include Sidekiq::Worker

      def perform(lead_bundle_subscription_id)
        Rails.error.handle(ActiveRecord::RecordNotFound) do
          ::Companies::LeadBundleSubscription.find(lead_bundle_subscription_id).pause!
        end
      end
    end

    class ActivateJob
      include Sidekiq::Worker

      def perform(lead_bundle_subscription_id)
        Rails.error.handle(ActiveRecord::RecordNotFound) do
          ::Companies::LeadBundleSubscription.find(lead_bundle_subscription_id).unpause!
        end
      end
    end

    class StartStopPauseJob
      include Sidekiq::Worker

      def perform
        ::Companies::LeadBundleSubscription.with_pause_start_date.in_the_past.find_each do |lead_bundle_subscription|
          ActivateJob.perform_async(lead_bundle_subscription.id)
        end

        ::Companies::LeadBundleSubscription.with_pause_start_date.in_the_future.find_each do |lead_bundle_subscription|
          PauseJob.perform_async(lead_bundle_subscription.id)
        end
      end
    end
  end
end
```