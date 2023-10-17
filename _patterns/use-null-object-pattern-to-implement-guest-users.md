---
categories: Ruby, Rails
name: Use null object pattern to simplify code around guest users
---

We have a method throughout our codebase - `current_user` - that returns the current user.

However, if the user is not logged in, this `current_user` is `nil`.

## Problems

This has a number of issues:

**1. Error prone.**

Calling any method on `nil` results in a crash.

So engineers must remember to use the safe navigator or use an `if current_user` guard clause around any code that uses `current_user`.

The design doesn't help us spot or prevent mistakes.

**2. Difficult to reason about**

Because of the hacks above, logged out behaviour is difficult to reason about.

There's no place in the code we can see how a guest user should behave.

**3. Lack of consistency**

We have methods to query about every aspect of a user - `#admin?`, `#moderator?` etc .

When it comes to what should be a simple `#guest?` method, we instead have to do hacky `if` statements like `if current_user`.

Every `if` results in an increase in cyclomatic complexity and an extra thing to test for.

**4. Increases complexity**

We need to do tricks like this:

```ruby
def user_id_or_anonymous_id
  user_id.presence || anonymous_id
end

def user_id
  current_user&.id.to_s
end
```

Specifically for tracking, the code is littered with `user_id` and `anonymous_id` being passed as two separate parameters when really they both belong to a user.

## Solution

Use an object to encapsulate the behaviour of a guest user.

**1. Add a Guest Social User model**

```ruby
class GuestSocialUser
  def admin? = false
  def regular? = false
  def guest? = true

  # For AR relations, `.none` gives back no objects
  def conversations = Conversation.none

  # Set the anonymous ID in memory
  def with_anonymous_id(anonymous_id)
    self.anonymous_id = anonymous_id
    self
  end
  # ... all other methods that social user responds to ...
end
```

**2. Create a `User.current` factory method**

```ruby
class User
  def self.current(auth_token:, cookies:)
    current_user = if auth_token
      # ... decode token and extract id ...
      SocialUser.not_deactivated.find_by(id: id_from_token) || GuestSocialUser.new
    else
      GuestSocialUser.new
    end
    current_user.with_anonymous_id(cookies[:ajs_anonymous_id])
  end
end
```

Simplify the `current_user` helper method:

```diff
 def current_user
-  return @_current_user if defined?(@_current_user)
-
-  @_current_user ||= SocialUser.not_deactivated.find_by id: user_data_from_token["id"]
+  @_current_user ||= User.current(auth_token:, cookies:)
 end
```

**3. Update all callers**

Now everywhere we have `current_user` we can depend on it being a user.

No more `if current_user.blank?`:

```diff
 class ConversationChannel < ApplicationCable::Channel
   def subscribed
-    return if current_user.blank?
     return unless (conversation = current_user.conversations.find_by(id: params.fetch(:id)))

     stream_for conversation
   end
 end
```

For tracking, we ditch `user_id` and `anonymous_id` in favour of `user`:

```diff
 class UserTrackingJob < ApplicationJob
   queue_as :within_five_minutes
 
-  def perform(user_id, event, properties = {}, context = {}, anonymous_id = "")
+  def perform(user, event, properties = {}, context = {})
     UserTracking.track(
-      user_id: user_id,
+      user:,
       event: event,
       properties: properties,
       context: context,
-      anonymous_id: anonymous_id,
     )
   end
 end
```

# Examples

## Bad

```ruby
  def login_required
    return if logged_in?

    respond_to do |format|
      format.json do
        render json: { errors: ["You must be logged in to do that"], not_logged_in: true }.to_json,
               status: :forbidden
      end
    end
  end
```

## Good

```ruby
  def login_required
    respond_with_forbidden if current_user.guest?
  end

  def respond_with_forbidden
    respond_to do |format|
      format.json do
        render json: { errors: ["You must be logged in to do that"], not_logged_in: true }.to_json,
              status: :forbidden
      end
    end
  end
```

## Bad

```ruby
if current_user
  track(current_user.id, "Account Request Created")
else
  track(nil, "Account Request Created", {}, {}, cookies[:ajs_anonymous_id])
end
```

Method definition:

```ruby
def track(tracking_id, event_name, attributes = {}, context = {}, anonymous_id = "")
  return if attributes[:skip_event]

  UserTrackingJob.perform_later(
    tracking_id,
    event_name,
    attributes,
    context_attributes.merge(context),
    anonymous_id.presence || Events::Context.anonymous_user_id_from_email(cookies: cookies),
  )
end
```

## Good

```ruby
track(current_user, "Account Request Created")
```

Method definition:

```ruby
def track(user, event_name, attributes = {}, context = {})
  return if attributes[:skip_event]

  UserTrackingJob.perform_later(
    user,
    event_name,
    attributes,
    context_attributes.merge(context),
  )
end
```

## Bad

```ruby
def most_recent_lead
  return nil if current_user.blank?

  Rails.cache.fetch("most_recent_lead_#{current_user}") do
    investor_leads.most_recent.first
  end
end

def investor_leads
  @_investor_leads ||= Marketplace::Businesses::Lead.where(investor: current_user)
end
```

## Good

```ruby
def most_recent_lead
  Rails.cache.fetch("most_recent_lead_#{current_user}") do
    current_user.investor_leads.most_recent.first
  end
end
```

```ruby
class SocialUser
  # Use a proper active record relation, not a memoized cached variable in the controller
  has_many :investor_leads 
end

class GuestSocialUser
  # Duck type the relation to return no objects
  def investor_leads 
    Marketplace::Businesses::Lead.none
  end
end
```

# Why?

See the problems above. Using the null object pattern for guest users will result in:

## 1. Less footguns

No more crashes when you call `current_user.investor_leads` and the user is logged out.

## 2. Simpler code at the call site

As seen above in the examples, we're pushing complexity into `GuestSocialUser` and `User`.

This results in simpler call site code around `current_user`.

## 3. Easier to reason about

There's an extra abstraction, but you only need to look in there when you want to understand how it works.

The details will be hidden behind the user duck type.

At the call sites the code will be simpler, so will be easier to reason about.

In addition, there will be one place in the codebase - `GuestSocialUser` - that will describe the behaviour of a logged out or guest user.

Currently this behaviour is scattered throughout the codebase on a case by case basis.

## 4. Improved consistency

Just pass around `user`. No need for special `if` cases that cover different edge cases.

Get user. Call method you want. One programming model means consistency.

## 5. Simplifies tracking

Currently we need to pass around `user_id` and `anonymous_id` over and over again.

This is a [Data Clump](https://sourcemaking.com/refactoring/smells/data-clumps) code smell.

By encapsulating both these methods into the user abstraction, all the tracking code would become radically simpler and easier to reason about.

# Drawbacks

* An extra model - `GuestSocialUser` - will attract significant extra complexity
* Wide reaching change across the codebase - this move would have to be done gradually over many PRs
* Another abstraction to reason about when debugging