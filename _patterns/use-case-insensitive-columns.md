---
categories: Rails
name: "Use Citext extension for Case InSensitive Columns"
---

## Problem

For the columns where we do not care about the capitalization of data inside it, using lower or downcase every where in our application can increase query time.

* Need to use "lower()" or ".downcase" everywhere the column(s) being used.
* Query time is high for fetching data and then downcasing it.

## Solution

The citext extension can provide several benefits for developers who need to work with case-insensitive text data in their database applications.

* Citext can be faster than using the lower or downcase functions, particularly for large datasets.
* Citext uses an index, which allows for faster searching and sorting of data.
* Citext is particularly useful when dealing with user input, where users may not be consistent with capitalization.
* It can help avoiding to deal with different variations of the same data.
* It allows users to input data in any case they prefer, without having to worry about matching existing data.

## Where do we use Citext?

1. **Add Migrations** - when defining a type for a new column, citext can be given as a type for that column.
2. **Updating Migrations** - when updating a type for an already existing column, citext can be given as a type for that column.

This way, lower or downcase can be removed and citext will autmatically compare case-insensitive data.

## How do I use Citext?

1. `t.citext :country` - when you need to define a column type in a migration.
2. `change_column :users, :email, :citext` - when you need to update the column type.

> Note: limit cannot be added to citext columns so use model-level validations to control the length of them.

### Bad
````ruby
user = User.find_by(email: "Testerson@gmail.com".downcase)
````

### Good
````ruby
class EnableCitextExtension < ActiveRecord::Migration[6.1]
  def change
    enable_extension "citext"
  end
end

class ChangeEmailInUsers < ActiveRecord::Migration[6.1]
  def change
    change_column :users, :email, :citext
  end
end

user = User.find_by(email: "Testerson@gmail.com")
````

### Bad
````ruby
user = User.order("lower(name)")
````

### Good
````ruby
class ChangeEmailInUsers < ActiveRecord::Migration[6.1]
  def change
    change_column :users, :name, :citext
  end
end

user = User.order("name")
````

## Further Reading

* [Citext use in Rails](https://www.mikecoutermarsh.com/storing-email-in-postgres-rails-use-citext/)
