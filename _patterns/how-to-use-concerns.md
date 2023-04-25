---
categories: Rails
name: Don't misuse modules and concerns
---

Because modules and concerns enable multiple-inheritance in Ruby, they can be a great way to share reusable
functionality between classes. However, they can be misused.

### Good

We should use concerns to:

- Share concrete, discrete functionality between classes. For example, a `HasNotifications` concern might be used to
  extend a `User` and `Company` class so that they both have the behavior for managing notifications.

```ruby
module HasNotifications
  extend ActiveSupport::Concern

  included do
    has_many :notifications, dependent: :destroy, as: :notified
  end
end

class User
  include HasNotifications
end

class Company
  include HasNotifications
end
```

### Bad

We should not use concerns to:

- Make a class "smaller", as this is purely cosmetic
- Avoid placing code at the correct layer of our MVC architecture
- Abstract logic that belongs in a PORO

```ruby
module UserExtensions
  module Notifications
    extend ActiveSupport::Concern

    included do
      has_many :notifications, dependent: :destroy
    end
  end
end

class User
  include UserExtensions::Notifications
end
```

## Further reading

- [About Rails Concerns](https://medium.com/@carlescliment/about-rails-concerns-a6b2f1776d7d)
