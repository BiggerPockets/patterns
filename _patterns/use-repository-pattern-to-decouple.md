---
categories: Ruby
name: Use Repository pattern to decouple persistence from domain logic
---

## The problem

The ActiveRecord pattern that Rails uses mixes two different concerns:

* Domain logic
* Persistence

## The symptoms

* Large classes
* Behaviour that's difficult to test
* Slow tests
* Lots of mocking and stubbing

## What's domain logic? What's persistence?

**Domain logic**

What the business cares about.

```ruby
  def post_score(time_frame = :this_week)
  end

  def registered_for_upcoming_webinar?
  end
```

**Persistence**

Reading or writing the object to a durable location.

```ruby
  scope :recently_visited, -> { }

  def deactivate
  end
```

## Why is this a problem?

These are different concerns.

By putting them together, we're giving the object more reasons to change.

* Domain logic is fast, persistence is slow
* Domain logic is business related, persistence is technology related

Fine... but how does this affect us?

## Real world problem

* We use an old version of the Stripe API
* We'd like to try out the latest API version
* We'd also like to upgrade in a controlled way

**The concerns**

* Persistence - storing and retrieving data from the Stripe API
* Domain logic - rules around upgrades, downgrades, synchronisation logic

**With our current design this is expensive**

**No Repositories**

```ruby
invoice = Stripe::Invoice.retrieve(id: invoice_id, expand: ["subscription"])     # Persistence
subscription = invoice.subscription                                          

raise BlankSubscription if subscription.blank?

return if subscription.status != "active"                                        # Domain

user = SocialUser.find(extract_social_user_id_from(subscription: subscription))  # Persistence / Domain
plan_id = Billing::ExtractPlanFromStripeSubscription.new.call(subscription)      # Domain

return if subscription_upgraded_downgraded_or_renewed?(user:, subscription:)
```

**With Repositories**

```ruby
# retrieve invoice
# raises BlankSubscription if there's no subscription
invoice = @invoice_repo.with_subscription!(id: invoice_id, include: :subscription)

return if invoice.subscription_inactive?

investor = @investor_repo.find(invoice.subscription_owner_id)

return if investor.upgraded? || investor.downgraded? || investor.renewed?
```

In the initializer:

```ruby
def initialize(
  invoice_repo: Membership::Billing::Stripe::InvoiceRepo.new,
  investor_repo: Membership::Billing::Postgres::InvestorRepo.new
)
  @invoice_repo = invoice_repo
  @investor_repo = investor_repo
end
```

Repositories:

```ruby
class Membership::Billing::Stripe::InvoiceRepo
  def with_subscription!(id:, include:)
    # ...
  end
end

class Membership::Billing::Postgres::InvestorRepo
  def find(id)
    SocialUser.find(id)
    # Pointless abstraction?
    # This allows us to:
    # * split SocialUser apart - it gives us a seam
    # * evolve persistence just for the `Membership::Billing` bounded context
    # * introduce the domain concept of an "Investor" gradually
  end
end
```

Domain objects:

```ruby
class Membership::Billing::Invoice
  def subscription_owner_id
  end

  def subscription_inactive?
  end
end

class Membership::Billing::Investor
  def upgraded?
  end

  def downgraded?
  end

  def renewed?
  end
end
```

**How could we upgrade Stripe?**

This helps to solve our original problem.

1. Introduce a new repository for the new API version:

```ruby
class Membership::Billing::StripeV2023_02_03::InvoiceRepo
  def with_subscription!(id:, include:)
    # ... 
  end
  # ... other query and command methods ...
end
```

2. Change code inside new repo to return objects that match interfaces already in use
3. Test the repo against the real API version using VCR
4. Swap out the repo - all at once, behind a feature flag or using Scientist
5. Get rid of original repo and rename current repo to without the version number

**How this helps testing**

For testing, we can dependency inject an **in memory** repository.

This would speed up the test suite at the expense of added complexity.

Given Rails' reliance on ActiveRecord, this repository pattern should be used carefully.

It may make sense to still have AR backed models write to the database and only use repositories for APIs.

## The cons

* Doesn't play nice with ActiveRecord
* Can lead to coupling across features
* More code
* Unfamiliar to Rails developers
* Need to match the behaviour and interfaces between repos **exactly**

## Mitigations

* Doesn't play nice with ActiveRecord - let Rails be Rails where it makes sense
* Can lead to coupling across features - limit repositories to bounded contexts
* More code - possibly use libraries for some reusable parts of repositories
* Unfamiliar to Rails developers - this document and workshops
* Match behaviour - unit testing, tools - Sorbet, Scientist

## The pros

* Allows swapping out different persistent mechanisms
* Faster test suites
* Plays nicely with bounded contexts
* Encourages evolving domain language
* Separates out concerns leading to more modular designs
