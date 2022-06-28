---
categories: Rails
name: Two step deploy when table is being dropped and references to it are being removed
---

It's common when we have to drop some colums of a table in a database or we have to even drop table. Those dropped columns or tables may be used in our application. We may not be aware of all the places where those dropped columns and tables are being used.

When we drop columns and tables from the database and those are being used in some part of the application, which crashes the application when we do single deploy.

The best way to address this issue is to introduce a 2-step deploy process with the steps below:

1. Make the code compatible with the migration you need to run. Also ignore column on the table.

2. Run the migration, and remove any code written specifically for it

Step 1:
````ruby

class User < ActiveRecord::Base
  self.ignored_columns = %w(first_name)
end

````
Step 2:
````ruby

class RemoveUsersUpdatedAtColumn < Gitlab::Database::Migration[2.0]
  def up
    remove_column :users, :first_name
  end

  def down
    add_column :users, :first_name, :string
  end
end
````
and remove ````self.ignored_columns = %w(first_name)```` from ````User```` model.
