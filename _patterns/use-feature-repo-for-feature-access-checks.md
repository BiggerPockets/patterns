---
categories: Ruby
name: Use feature repo for feature access checks
---

## Context

* We had lots of code inside controllers and views checking for methods like `#has_premium_benefits?` on user
* This code determines if the user has access to a **feature**

## Problem

* When a change is made to features or how access is granted, we need to make many changes throughout the codebase
* We cannot at a glance see which features the app contains, or which subscription levels have access to them
* The concept of a "feature" is missing from the code so it doesn't communicate intent as clearly as it could do

## Solution

* There's a single place for all feature configuration to be stored - [the `FeatureRepo`](https://github.com/BiggerPockets/biggerpockets/blob/86ee130dce1a272dd5ad59689ed4226de24bac89/lib/features/feature_repo.rb#L10)
* Features are named using a namespace
* Features are configured with a `:subscription_level`, `:enabled` or `:value`
* Features can be checked with `FeatureRepo#enabled?` or `FeatureRepo#value`

## Bad

```ruby
class WebinarBonusesController < ApplicationController
  before_action :require_annual, only: :create

  def create
    # ...
  end

  def require_annual
    return if current_user.paid_annual? || current_user.has_premium_benefits?

    # ...
  end
  # ...
end
```

Taken from the code [as it used to be](https://github.com/BiggerPockets/biggerpockets/blob/eadc8aa5cf73b2d697a12b9cdff28f2421e0ca6e/app/controllers/webinar_bonuses_controller.rb#L41-L46).

## Good

```ruby
class WebinarBonusesController < ApplicationController
  before_action :require_annual, only: :create

  def create
    # ...
  end

 def require_annual
    return if feature.enabled?("membership.signup.access_to_webinar_bonuses", user: current_user)

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
    "membership.signup.access_to_webinar_bonuses" => [ # <== 
      { subscription_level: :premium, enabled: true },
      { subscription_level: :pro, subscription_period: :annual, enabled: true },
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
      "membership.signup" => "Membership signup",
      # ...
    }
  end
end
```
