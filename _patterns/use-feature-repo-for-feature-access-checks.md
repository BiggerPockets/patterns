---
categories: Ruby
name: Describe features at a high level rather than querying the user model directly
---

## Context

* Lots of code inside controllers and views that make specific queries like `#has_paid_subscription?` to user
* This code is really checking if the user has access to a feature

## Problem

* Code is brittle - one change to how subscriptions work mean shotgun surgery on the codebase
* Difficult to reason about features - we can't see in one place the features of the app or which subscription levels are granted access
* Code doesn't communicate intent - the concept of a "feature" is missing

## Solution

* Features are named using a namespace
* Features are configured with a `:subscription_level`, `:enabled` or `:value`
* Features can be accessed with `feature.enabled?`, `feature.disabled?` or `feature.value`

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
