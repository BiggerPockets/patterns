---
categories: Rails
name: Prefer CRUD actions in controllers
---

In a Rails application, controllers typically represent a _resource_ type. In a RESTful design, the only actions an API consumer should be able to perform on a resource are: reading, creation, modification, and deletion. Rails usually will map these actions to HTTP verb -> method name pairs:

* GET / -> #index (collection)
* GET /new -> #new (collection)
* GET /123 -> #show (singular)
* GET /123/edit -> #edit (singular)
* DELETE /123 -> #destroy (singular)
* PATCH /123 -> #update (singular)
* POST / -> #create (collection)

If you find yourself needing to do anything else to a resource, consider two things:

1. Does this action actually fit into one of the other actions?
1. Am I in fact working with a different type of resource altogether?


## Example 1

### Bad

````ruby
class PostsController < ApplicationController
  # PUT /posts/1/archive
  def archive
    @post = Post.find(params[:id])
    @post.update(archive: true)
  end
end
````

## Good

````ruby
class PostsController < ApplicationController
  # PUT /posts/1
  # { post: { archived: true } }
  def update
    @post = Post.find(params[:id])
    @post.update(post_params)
  end
end
````
## Example 2
### Bad

````ruby
class UsersController < ApplicationController
  # POST /users/reset_password
  def reset_password
    current_user.reset_password
  end
end
````

### Good

````ruby
class PasswordResetsController < ApplicationController
  # POST /password_resets
  def create
    current_user.reset_password
  end
end
````

When applying this rule, consider the mental overhead of extracting a new resource out of a controller. For example, if
you want to give the user the ability to delete everything in a collection, it may make more sense to keep similar
logic near the other actions. In this case, it's OK to declare colletion resource routes such as `create_all`,
`update_all`, and `delete_all`.

### Bad

````ruby
class NotificationsController < ApplicationController
  def index
    # ...
  end
end

class AllNotificationsController < ApplicationController
  def destroy
    # operation to destroy all notifications
  end
end
````

### Good

````ruby
class NotificationsController < ApplicationController
  def index
    # ...
  end

  def destroy_all
    # operation to destroy all notifications
  end
end
````

This does not mean that you should willy-nilly add related routes that are not resourceful, even if the route
would affect the same resource:

### Bad

```ruby
class NotificationsController < ApplicationController
  # POST /notifications/:id/read
  def read
    # mark a notification as read
  end
end
```

### Good

It's probably best to simply handle this via the #update action, but if it absolutely has to be a separate route, here's what should be done:

```ruby
class Notifications::ReadsController < ApplicationController
  # POST /notifications/:id/read
  def create
     # mark a notification as read
  end
end
```
