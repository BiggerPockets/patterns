---
categories: Practices
name: Running a data migration scripts
---

## Objective

The primary objective is to establish a systematic approach for the efficient management and execution of data migration scripts, 
minimizing errors and reducing disruption for fellow developers.

## Execution of Scripts

When considering the execution of scripts, a general rule is to always use rake task apart from rare exception discussed below.
Adhere to the following guidelines:

- **Non Trivial Script Execution:**
  - If you have script that needs to be run, potentially multiple times:
    - Send it to rake task under the one-off migration section (`/lib/tasks/one_off/migrations.rake`).
- **Quick Small Changes in Database Records:**
  - For quick, small changes in database records:
    - What qualifies as small change?
      - attribute change to a single record
    - Access the console and make the necessary changes
    - Maintain an audit trail to understand why and what changes were made (https://github.com/basecamp/audits1984)

## How to create & run my script
To create and run your script, follow these steps:

- **Write Your Script as a Rake Task**
  - Create a class within migration directory script (eg. `/lib/migrations/do_x.rb`)
  - Add your task to list of one_off tasks in the file (`/lib/tasks/one_off/migrations.rake`)
  - eg.
    ```ruby
    namespace :one_off do
      namespace :migrations do
        ...

        desc "This tasks does x"
        task do_x: :environment do
          Migrations::DoX.new.call
        end
      end
    end
    ```
- **Run Your Rake Task**
  - Execute your rake task directly by establishing an SSH connection to the Heroku server
    - eg. `heroku run bash --app biggerpockets` and run the rake task `rake one_off:migrations:do_x`
  - Alternatively, run rake task in detached mode usually for long running tasks (https://devcenter.heroku.com/articles/one-off-dynos#running-tasks-in-background)
    - eg. `heroku run:detached rake one_off_migrations:do_x`
- **Remove The Rake Task and Class**
  - After executing the rake task in a production environment and is no longer necessary, 
  the engineer who implemented it is accountable for its removal. 

## Best Practices for Writing Data Migration Scripts
Follow [these]({{ site.baseURL }}/practices/best-practices-on-writing-a-script) general good practices when writing a data migration scripts.
