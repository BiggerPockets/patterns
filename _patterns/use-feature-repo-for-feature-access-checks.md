---
categories: Ruby
name: Use feature repo for feature access checks
---

## Context

* Lots of code inside controllers and views that make specific queries like `#has_paid_subscription?` to user
* This code is really checking if the user has access to a feature

## Problem

* Code is brittle - one change to how subscriptions work mean shotgun surgery on the codebase
* Difficult to reason about features - we can't see in one place the features of the app or which subscription levels are granted access
* Code doesn't communicate intent - the concept of a "feature" is missing

## Solution

* There's a single place for all feature configuration to be stored - [the `FeatureRepo`](https://github.com/BiggerPockets/biggerpockets/blob/86ee130dce1a272dd5ad59689ed4226de24bac89/lib/features/feature_repo.rb#L10)
* Features are named using a namespace
* Features are configured with a `:subscription_level`, `:enabled` or `:value`
* Features can be checked with `FeatureRepo#enabled?` or `FeatureRepo#value`

## Bad

```ruby
class PaidReportsController < ApplicationController
  before_action :require_paid, only: :create

  def create
    # ...
  end

  def require_paid
    return unless current_user.has_paid_subscription? && current_user.investor?

    # ...
  end
  # ...
end
```

## Good

```ruby
class PaidReportsController < ApplicationController
  before_action :require_paid, only: :create

  def create
    # ...
  end

 def require_paid
    return if feature.disabled?("membership.reporting.report_access_allowed", user: current_user)

    # ...
  end
  # ...
end
```

Then in the `Features` module:

```ruby
# lib/features/feature_repo.rb

module Features
  FEATURES = {
    # ...
    "membership.reporting.report_access_allowed" => [
      { subscription_level: :premium, enabled: true },
      { subscription_level: :none, enabled: false }
    ],
    # ...
  }
  # ...
end
```

You may need to add the app name:

```ruby
# lib/events/event.rb

module Events
  class Event < Dry::Struct
    # ...
    APP_NAMES = {
      "membership.reporting" => "Membership Reporting",
      # ...
    }
  end
end
```
