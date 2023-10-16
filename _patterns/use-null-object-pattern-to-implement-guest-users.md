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

So we must remember to use the safe navigator or use an `if current_user` guard clause around any code that uses `current_user`.

The code doesn't help us prevent mistakes.

**2. Encourages helper methods**

In [`Api::V1::BaseController#current_user`](https://github.com/BiggerPockets/biggerpockets/blob/aec6663c398a3e5b39e06caf4065e4fdde30a632/app/controllers/api/v1/base_controller.rb#L49-L53):

```ruby
def current_user
  return @_current_user if defined?(@_current_user)

  @_current_user ||= SocialUser.not_deactivated.find_by id: user_data_from_token["id"]
end
```

We also have two methods - `#logged_in?` and `#logged_out?` in [an `Authentication` concern](https://github.com/BiggerPockets/biggerpockets/blob/aec6663c398a3e5b39e06caf4065e4fdde30a632/app/controllers/concerns/authentication.rb#L92-L99) that are used in 400+ places through the codebase:

```ruby
def logged_in?
  !!current_user
end

def logged_out?
  !logged_in?
end
```

These methods belong on the user model, not globally accessible helpers.

**3. Difficult to reason about**

Because of the hacks above, logged out behaviour is difficult to reason about.

There's no place in the code we can see how a logged out user should behave.

**4. Lack of consistency**

We have methods to query about every aspect of a user - `#admin?`, `#moderator?` etc .

When it comes to what should be a simple `#guest?` or `#logged_in?` method, we can't and instead have to rely on the helpers above.

**5. Special case code for tracking**

We need to do tricks like this:

```ruby
def user_id_or_anonymous_id
  user_id.presence || anonymous_id
end

def user_id
  current_user&.id.to_s
end
```

This adds complexity to the tracking code.

## Solution

Use an object to encapsulate the behaviour of a guest user.

**1. Add a Guest Social User model**

```ruby
class GuestSocialUser
  def admin? = false
  def logged_in? = false
  def logged_out? = true
  def tracking_id
    # Code to get the anonymous ID for Segment
  end
  # ... all other methods that user responds to ...
end
```

**2. Update `current_user` to return a `GuestSocialUser`**

For example:

```diff
 def current_user
-  return @_current_user if defined?(@_current_user)
-
-  @_current_user ||= SocialUser.not_deactivated.find_by id: user_data_from_token["id"]
+  @_current_user ||= SocialUser.not_deactivated.find_by(id: user_data_from_token["id"]) || GuestSocialUser.new
 end
```

Or we could create a class method to further encapsulate:

```ruby
class SocialUser
  def self.from_auth_token(auth_token)
    if auth_token
      # ... decode token and extract id ...
      not_deactivated.find_by(id: id_from_token) || guest
    else
      guest
    end
  end

  def self.guest
    GuestSocialUser.new
  end
end
```

Then:

```diff
 def current_user
-  return @_current_user if defined?(@_current_user)
-
-  @_current_user ||= SocialUser.not_deactivated.find_by id: user_data_from_token["id"]
+  @_current_user ||= SocialUser.from_auth_token(auth_token)
 end
```

**3. Update all callers**

Some examples:

```diff
-if logged_in?
+if current_user.logged_in?
```

```diff
-if current_user
+if current_user.logged_in?
```

```diff
 class ConversationChannel < ApplicationCable::Channel
   def subscribed
-   return if current_user.blank?
     return unless (conversation = current_user.conversations.find_by(id: params.fetch(:id)))

     stream_for conversation
   end
 end

 class GuestSocialUser
   # ... existing code ...
+  def conversations
+    Conversation.none
+  end
 end
```

# Example

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
    return if current_user.logged_in?

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

# ... somewhere else ...

if current_user
  track(current_user.id, "...", {}, {}, "")
else
  track(nil, "...", {}, {}, cookies[:ajs_anonymous_id])
end
```

## Good

```ruby
def track(user, event_name, attributes = {}, context = {})
  return if attributes[:skip_event]

  UserTrackingJob.perform_later(
    user.id,
    event_name,
    attributes,
    context_attributes.merge(context),
    user.anonymous_id,
  )
end

# ... use this method ...

track(
  current_user,
  "Viewed Calculator PDF",
  "calculator_type" => "Rehab Estimator",
  "id" => params[:id],
)

# ... in the base controller ...
def current_user
  @_current_user ||= SocialUser.from(auth_token:, anonymous_id: cookies[:ajs_anonymous_id])
end

# ... in the SocialUser ...
class SocialUser
  # Factory method to encapsulate complexity of finding a user
  def self.from(auth_token:, anonymous_id:)
    anonymous_id = anonymous_id.presence || Events::Context.generate_anonymous_id
    if auth_token
      # ... decode token and extract id ...
      user = not_deactivated.find_by(id: id_from_token)
      if user
        user.anonymous_id = anonymous_id  # in memory attribute on social user
        user
      else
        guest(anonymous_id:)
      end
    else
      guest(anonymous_id:)
    end
  end

  def self.guest(anonymous_id:)
    GuestSocialUser.new(anonymous_id:)
  end
end


```