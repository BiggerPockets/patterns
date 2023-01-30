---
categories: Ruby
name: Use call for classes containing pure behaviour
---

## Problem

* There are no clear semantics in our codebase for objects that contain pure behaviour
* The limited conventions we do have are tightly coupled to superclasses
* They are also limiting for future flexibility and true object oriented design

## Example - [`GetLocalCitySuggestionService`](https://github.com/BiggerPockets/biggerpockets/blob/8b2914fdf835bc24c47a33e773caf1a939ef2edd/app/services/get_local_city_suggestion_service.rb/#L3)

* Consider the [`GetLocalCitySuggestionService` class](https://github.com/BiggerPockets/biggerpockets/blob/8b2914fdf835bc24c47a33e773caf1a939ef2edd/app/services/get_local_city_suggestion_service.rb/#L3)
* This object has one method - `#perform!`
* A dead giveaway that this object deals with **pure behaviour and no data**

## Use `#call` for pure behaviour

Why?

### 1. Idiomatic Ruby

* A lambda is for **pure behaviour**
* A lambda responds to `#call`
* The idiom for pure behaviour in Ruby is therefore `#call`

### 2. Gives future design options

* Any object that has `#call` can now duck type for a lambda
* This allows us to swap it out for a lambda in future and client code will Just Work

## Dependencies - static into `.new`, transient into `#call`

### What's a static dependency?

* Initialized once for the lifecycle of the app
* Examples: repos, config, setup

### What's a transient dependency?

* Object that is being "operated on"
* Every time the object is called, the transient dependency can change

Still unclear? Read [the analogy at the end of this document.](#analogy)

### Why?

* By injecting the static dependencies into `.new`, we can build the object once
* We call `#call` on the object with the transient dependencies

**At this point, we've made our object swappable**

This gives us design flexibility - just inject a different object that adheres to the same interface, and blam - you've got different behaviour.

# Examples

## Bad

```ruby
class GetLocalCitySuggestionService < ServiceObject
  def initialize(lat:, lng:)
    @lat = lat
    @lng = lng
  end

  def perform!
    # ... snip ...
  end
end
```

## Better

```ruby
class GetLocalCitySuggestionService
  def initialize(lat:, lng:)
    @lat = lat
    @lng = lng
  end

  def call
    # ... snip ...
  end
end
```

## Best

```ruby
class GetLocalCitySuggestionService
  def initialize(static:, dependencies:, here:)
    # ... snip ...
  end

  def call(lat:, lng:) # change on every call, so they're transient dependencies
    # ... snip ...
  end
end
```

## Using the object

### Bad & Better

```ruby
GetLocalCitySuggestionService.new(lat: latitude, lng: longitude).perform

# or

GetLocalCitySuggestionService.new(lat: latitude, lng: longitude).call
```

Challenge - what if we want to run the real suggestion service in prod, but a dummy one in test?

The only consistent interface is at the class level - the `.new`. So we'd have to swap out a different class:

```ruby
# test.rb
Rails.configure do
  city_suggestion_service_class = DummyLocationService
end

# production.rb
Rails.configure do
  city_suggestion_service_class = GoogleMapsSuggestionService
end
```

But... the Google maps suggestion service needs an API key. Hmmm...

At this point we need to resort to setting class level variables on `GoogleMapsSuggestionService`:

```ruby
GoogleMapsSuggestionService.api_key = ENV.fetch('GOOGLE_MAPS_API_KEY')
```

Yuck. This is now a global variable that can be mutated by any thread. 

Can we do better?

## Best

Want to have the real suggestion service for production, but a dummy one for test mode?

No worries! Because we've got our dependencies defined correctly and we've got a consistent interface, we can:

```ruby
# test.rb
Rails.configure do
  city_suggestion_service = ->(lat:, lng:) { [{ name: "Dummy Location", postcode: "AB123 456" }]} # Use a lambda no problem!
end

# production.rb
Rails.configure do
  city_suggestion_service = GoogleMapsSuggestionService.new(api_key: ENV.fetch('GOOGLE_MAPS_API_KEY')) # Or use a fully fledged object
end
```

Because the transient dependencies are in the right place, we get swappability:

```ruby
Rails.configuration.city_suggestion_service.call(lat: latitude, lng: longitude)
```

## Analogy

Imagine you've hired a graduate to enter data from paper documents into Google sheets.

You set them up in your office - give them a desk, a chair, a laptop, an internet connection.

Once they're set up, you start feeding them paper documents.

For each document, you put it in their inbox. They type it into Google sheets.

You have an established pattern to give them work.

Life is good. 

### Now to scale...

You hire another graduate. You set them up differently.

They maybe prefer a Windows laptop. They might prefer a different type of chair (they've got a bad back).

However, you make the system exactly the same - an inbox on their desk. You give them a document. They type it into Google sheets.

You can now drop these sheets into either inbox, safe in the knowledge that the graduate will do the work.

### Principles

**Static dependencies** are the desk, the chair, the laptop, the internet connection. They're everything they need to get the job done.

**Transient dependencies** are the paper documents you give to them. Each one is different. Of course, they still need these documents to get the job done, but they change on every operation.

**Consistent interface** the interface is the inbox on each graduate's desk. It's the contract you've got saying "when you see a document in this inbox, please type it into Google sheets".

Because you've defined the dependencies correctly and you have a consistent interface, **this allows you to scale**.

If you had 20 graduates with this system, you'd be flying.

### Good object oriented design

This is what good object oriented design looks like.

You've set up objects with everything they need. Each graduate has different static dependencies.

But because these static dependencies are set up **once and once only**, as a manager, you don't have to put daily energy into this.

Heck, because you've defined the exact interface you want (Google sheets and the inbox), you could now easily **hire someone** to set each graduate up.

What would you say to them?

> Please make sure the graduate has an inbox and access to Google sheets. I don't care about how.

I think you'd agree that **this is a good manager**.

They define some expectations and let everyone decide how best to do it.

### Mixing up static and transient dependencies

Imagine if as a manager, you decided that for every document, you had to set up the chair, the desk, the laptop.

You gave the graduate the document in their inbox, they typed it in.

Then you took away the chair, the desk, the laptop.

For the next document you repeated the whole rigmorale again.

You'd have to micromanage every single graduate. Hand hold them setting up their working space.

This would be unsustainable, even for one graduate. And it's patently absurd.

Again, I think you'd agree that **this is a terrible manager**.

Can you imagine what would happen as this company scaled?

A company with 20 graduates where the manager sets their laptops up from scratch every time they do some work?

It'd be utter chaos!

**And yet in software we do this all the time.**

