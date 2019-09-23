# Restrict controllers to CRUD actions

In a Rails application, each controller should represent a _resource_ type. In a RESTful design, the only actions an API consumer should be able to perform on a resource are: reading, creation, modification, and deletion. Rails usually will map these actions to HTTP verb -> method name pairs:

* GET / -> #index (collection)
* GET /new -> #new (collection)
* GET /123 -> #show (singular)
* GET /123/edit -> #edit (singular)
* DELETE /123 -> #destroy (singular)
* PATCH /123 -> #update (singular)
* POST / -> #create (collection)

If you find yourself needing to do anything else to a resource, consider two things:

  a) Does this action actually fit into one of the other actions?
  b) Am I in fact working with a different type of resource altogether?

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
