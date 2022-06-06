---
categories: Rails
name: Use ActiveJob
---

When writing a background task, prefer ActiveJob over Sidekiq::Worker.

Benefits of ActiveJob over Sidekiq::Worker:
- Advanced serialization and deserialization provided by Rails (GlobalID), allowing us to pass models in as arguments, rather than just IDs.
- It is easier to switch to a different job processing backend.

Use [`sidekiq_options`](https://github.com/mperham/sidekiq/wiki/Active-Job#customizing-error-handling) if you need more nuanced Sidekiq-specific configuration.

## Bad

````ruby
# app/workers/webinar_calendar_invitation_worker.rb

class WebinarCalendarInvitationWorker
  include Sidekiq::Worker
  sidekiq_options queue: :low

  def perform(webinar_join_event_id)
    webinar_join_event = WebinarJoinEvent.find(webinar_join_event)
    
    WebinarCalendarMailer.invite(webinar_join_event).deliver_now
  end
end

````

## Good

````ruby
# app/jobs/webinar_calendar_invitation_job.rb

class WebinarCalendarInvitationJob < ApplicationJob
  def perform(webinar_join_event)
    WebinarCalendarMailer.invite(webinar_join_event).deliver_now
  end
end
````
