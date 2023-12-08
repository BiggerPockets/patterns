---
categories: Practices
name: Best practices on writing a script
---

Follow these general practices when writing a regular or a data migration script:

- **Track Progress:**
  - Keep a record of the progress made during the execution of the script
- **Record Before and After States:**
  - Document the states of the data before and after script execution
- **Implement Dry Run Mode:**
  - Include a dry run mode in your script to allow running it without making actual changes to database records, 
    providing a preview of the run. It helps us realize the extent of the actual change the script will make.
  - eg.
    ```ruby
    class DoX
      def initialize(dry_run: true)
        @dry_run = dry_run
        ...
      end

      def call
        if @dry_run
          ## just log the changes it will do
        else
          ## make the actual database change
        end
      end
    end
    ```
