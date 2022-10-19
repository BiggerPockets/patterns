---
categories: Rails
name: Avoid creating static references to record instances
---

When using an ORM, be careful when using static (class-level) references to record instances.

## What is a "static reference to a record instance"?

Here's an example of a static reference to a record instance:

```ruby
class Category < ApplicationRecord
  def self.agent_category
    find_by(name: "Real Estate Agent")
  end
end
```

## Why is this dangerous?

This example assumes that there will only ever be one record with this name, which can be easily overlooked:

```ruby
Category.create(name: "Real Estate Agent")
Category.create(name: "Real Estate Agent") # Category.agent_category is now non-deterministic
```

Great care must be made so that the record's name never changes:

```ruby
cat_a = Category.create(name: "Real Estate Agent")
cat_b = Category.create(name: "Real Estate Brokerage")
cat_b.update(name: "Real Estate Agent") # Category.agent_category is now non-deterministic
```

Rails assumes that your app and tests are thread safe, but static methods are not thread safe.

```ruby
class Category < ApplicationRecord
  def self.agent_category
    find_or_create_by(name: "Real Estate Agent")
  end
end

Thread.new { Category.agent_category }
Thread.new { Category.agent_category } # Category.agent_category is now non-deterministic. There may be duplicate records.
```

## What is a safer alternative?

If your application assumes that something must exist, then it should be represented as code rather than as a record in the database. There are several techniques that are a safer alternative:

### ActiveRecord's enum

```ruby
class Company < ApplicationRecord
  enum category: { agent: 1, ... }
end
```

### ActiveHash

```ruby
class Category < ActiveHash::Base
  field :name

  add id: "real-estate-agent", name: "Real Estate Agent"
end
```

### Validations and Database Constraints

```ruby
class AddUniquenessConstraintToCategory < ActiveRecord::Migration
  def change
    add_index :categories, :name, unique: true
  end
end

class Category < ApplicationRecord
  def self.agent_category
    find_or_create_by(name: "Real Estate Agent")
  end

  validates :name, uniqueness: true
end
```
