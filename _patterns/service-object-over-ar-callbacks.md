---
categories: Rails
name: Do not use ActiveRecord callbacks
---

Unexpected side effects should be avoided. They introduce tight coupling between the data model and the system log. Additionally, it is difficult to opt-out of running callbacks when interacting with models.

A good alternative is to create a service object that is specific to the case where the callback is needed.

## Bad

````ruby
class BlogPost < ActiveRecord::Base
  belongs_to :blog
  after_create :notify_subscribers

  private
  def notify_subscribers
    blog.subscribers.each do |subscriber|
      BlogPostNotificationMailer.deliver(id, subscriber.id)
    end
  end
end
````

## Good

````ruby
class BlogPost < ActiveRecord::Base
  belongs_to :blog
end

class BlogPostCreationService
  def call(blog, blog_post_params)
    ActiveRecord.transaction do
      blog_post = blog.blog_posts.create(blog_post_params)

      blog.subscribers.each do |subscriber|
        BlogPostNotificationMailer.deliver(blog_post.id, subscriber.id)
      end
    end
  end
end
````
