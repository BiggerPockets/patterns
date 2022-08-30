---
categories: Rails
name: Validate correctly on the edge of the app
---

We should validate data on the edges of the app in controllers using `dry-validation` and `dry-types`. Invalid states should not be allowed.

## Bad - no validation

* Allows bad data in database - data quality is poor so analysis cannot be done
* If dependencies assume the data is good, apparently unrelated code blows up days, weeks or months later, leading to difficult to diagnose defects
* If dependencies check the validity of the data, we get hundreds of lines of boilerplate showing up all over the app and we still can't recover and have to crash anyway
* Sometimes we rely on JS to supply valid data to the backend - bad idea since JS is notoriously unreliable
* This kind of defect would be uncovered by using thorough unit tests, which implies testing is poor or non existent

## Good - precise validation at the edge

* Bad data should never be allowed into the app or the database
* Validation should happen at the edge - see Hexagonal Architecture literature for rationale
* For Rails, that means at the controller level
* Use `dry-validation` library for precise validation

## Why `dry-validation`?

Another gem just to do validation? It's worth it:

1. This gem is [way more flexible and feature filled than the sloppy and tightly coupled validations in `ActiveModel`](https://dry-rb.org/gems/dry-validation/0.13/comparison-with-activemodel/)
2. By separating out the validation concern it means [we are pushed towards a world where we cannot instantiate an invalid object](https://solnic.codes/2015/12/28/invalid-object-is-an-anti-pattern/) - this avoids a **huge** Rails anti-pattern that is the root cause of this crash
8. It plays really nicely with `Dry::Types` allowing us [to define required enums concisely](https://github.com/BiggerPockets/biggerpockets/pull/15580/files#diff-0fcbcb67d22b51f513a7a84c60f4e87b917a1cb4a5fb2ee1612782cd03333c32R9)
9. Allows us to [inject other objects to validate with](https://dry-rb.org/gems/dry-validation/1.8/external-dependencies/). So in future we can plug in `US::City` and so validate the city exists in the database. Not done in this PR but it'd be a big step forwards!
10. Like `ActiveModel` it [seamlessly supports translation of error messages](https://dry-rb.org/gems/dry-validation/1.8/messages/#using-localized-messages-backend) for when we need to show errors to users
11. It [separates out validation from the domain object](https://dry-rb.org/gems/dry-validation/1.8/#quick-start) giving a result object that responds to `#success?` and `#failure?`.
12. We can [call `#to_h` on the validation result to give a hash](https://github.com/BiggerPockets/biggerpockets/pull/15580/files#diff-9c31cee42c413d70d394d21d52d5c6371f6fdcd0081d2650bb8141aa1da87c21R16) we can use to build the `Dry::Struct` seamlessly, cutting down on messy boilerplate code.


## Example

### Bad - zero validation, sloppy testing, timebomb code

* Accept any location then later app will blow up
* [`Agent::Match::FlowsController#index`](https://github.com/BiggerPockets/biggerpockets/blob/9528e528f4e6dcdf045f2eb854bb03ac1d18fb8b/app/controllers/agent/match/flows_controller.rb/#L8-L15) shows the anti-pattern:

```ruby
module Agent
  module Match
    class FlowsController < BaseController
      def index
        @location = params[:location] # no validation, accepts any location
        # ... snip ...
      end
    end
  end
end
```

## Good - precise validation, no bad data in database, shows user error on invalid data

```ruby
module Agent
  module Match
    module Location
      # Describes a state using an enum - cannot be invalid
      State = Dry.Types::String.enum(
        'ALABAMA' => 'AL',
        'CALIFORNIA' => 'CA',
        # ... snip ...
      ).constructor(&:upcase)

      # Describes a city - currently too loose - will look up city/state combo in db
      City = Dry.Types::String.constructor(&:titleize)

      # Validate defines the validation rules
      class Validate < Dry::Validation::Contract
        params do
          required(:city).value(City).filled
          required(:state).value(State).filled
        end
      end

      # Raw converts from "Denver, CO" -> { city: "Denver", state: "CO" }
      class Raw
        def self.from_display_location(location)
          raw_city, raw_state = location.split(",").map(&:strip)
          { city: raw_city, state: raw_state }
        end
      end
    end

    # Finally, protect against bad data at the controller level
    class FlowsController < BaseController
      def index
        raw_location = Location::Raw.from_display_location(params[:location])
        validation = Location::Validate.new.call(raw_location)
        if validation.success?
          @location = validation.to_h # { city: City.new('Denver'), state: State.new('CO') }
                                      #   ^^^ A hash with validated instances
                                      #       At this point, location is valid so can be depended on
          # ... snip ...
        else
          redirect_to root_path, alert: "Invalid location #{location}" # Show error if location is invalid
        end
      end
    end
  end
end

```