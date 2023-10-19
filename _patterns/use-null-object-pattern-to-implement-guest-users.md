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

New engineers must learn that `current_user` can be `nil`.

Engineers must also remember to use the safe navigator or `if current_user` guard clause around any code that calls methods on `current_user`.

The design doesn't help us spot or prevent mistakes.

**2. Difficult to reason about**

Logged out behaviour is difficult to reason about because the logic is spread throughout the app in the `if` statements and safe navigators.

There's no place in the code we can see how a guest user should behave.

**3. Lack of consistency**

We have methods to query permission levels of a user - `#admin?`, `#moderator?` etc.

Instead of using `#guest?`, we need to rely on `if` statements as mentioned above.

Every `if` results in an increase in cyclomatic complexity and an extra thing to test for.

**4. Increases complexity**

When tracking, we have `user_id` and `anonymous_id` as two separate objects.

So we need code such as:

```ruby
def user_id_or_anonymous_id
  user_id.presence || anonymous_id
end

def user_id
  current_user&.id.to_s
end
```

The anonymous ID and user ID are two separate attributes of the user.

We need special cases to handle user ID and anonymous IDs for guest users.

In addition, without a guest user abstraction, it can be tempting to merge the user ID and anonymous ID as above, which is incorrect [^1].

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

    respond_with_forbidden
  end
```

## Good

```ruby
  def login_required
    respond_with_forbidden if current_user.guest?
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

## Good

```ruby
track(current_user, "Account Request Created")
```

## Bad

```ruby
def most_recent_lead
  return nil if current_user.blank?

  investor_leads.most_recent.first
end

def investor_leads
  @_investor_leads ||= Marketplace::Businesses::Lead.where(investor: current_user)
end
```

## Good

```ruby
def most_recent_lead
  current_user.investor_leads.most_recent.first
end

class GuestSocialUser
  # Duck type the relation to return no objects
  def investor_leads 
    Marketplace::Businesses::Lead.none
  end
end
```

# Why?

Using the null object pattern for guest users will result in:

## 1. Less footguns

No more crashes if calling `current_user.some_method` and the user is logged out.

## 2. Simpler code at the call site

Pushing complexity into `GuestSocialUser` and `User` means simpler call site code around `current_user`.

The cyclomatic complexity of the `if current_user` would be pushed into one place - inside the factory method.

## 3. Easier to reason about

One class - `GuestSocialUser` - describes the behaviour of a logged out or guest user.

If an engineer wants to understand the behaviour of a guest user, they have one class to look in.

This might also make the feature access checks code easier to reason about too.

## 4. Improved consistency

Pass around `user`. No need for special `if` cases that cover different edge cases.

One programming model means consistency.

## 5. Simplifies tracking

Currently we pass around `user_id` and `anonymous_id` together.

This is a [Data Clump](https://sourcemaking.com/refactoring/smells/data-clumps) code smell.

By encapsulating both these methods into the user abstraction, all the tracking code would become simpler and easier to reason about.

# Drawbacks

* An extra model - `GuestSocialUser` - will attract significant extra complexity
* Wide reaching change across the codebase - this move would have to be done gradually over many PRs
* Another abstraction to reason about when debugging

[^1]: A logged in user should have both a user ID and anonymous ID, whereas a guest user just has anonymous ID. However, they should always be sent to Segment as two separate fields, otherwise users will be incorrectly identified.