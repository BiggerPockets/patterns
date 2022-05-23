---
categories: RSpec
name: Avoid using let if possible
---
There are a few features of RSpec's DSL that are frequently overused, leading to an increase in test maintenance and a decrease in test readability.
`let` and `let!` are one of them when it comes to our codebase. It gets even worse, when spec grows larger and we have nested `let` being used within the spec scenarios.

This approach causes a number of issues:
- Obscures each test by introducing a Mystery Guest. Basically hampers the readability of spec.
- It causes fragile tests by creating a complicated fixture that is difficult to maintain.
- It causes slow tests by creating more data than is necessary in each test.

Benefits of avoiding let:
- Test scenarios are easier to read, because all the actors referenced from the verification step are declared in the setup step within the it block.
- They're less brittle, because each example only specifies the information it needs.
- They're faster, because each example is running with a smaller data set.

Hence, unless absolutely necessary, or there is a good reason to use it, avoid DSL constructs like subject, let, let!. 
It is fine to introduce repeated lines of code/duplication in spec, if it helps in readability. Stick to your old friend variables 
instead of let so that specs are easy to read and reason.

## Bad

```ruby
subject { described_class.new(search_location: search_location).fetch }
let!(:featured_premium_agent) do
  create(
    :company_profile,
    :premium,
    :agent,
    :featured,
    lat: denver_city.lat,
    lng: denver_city.lng,
  )
end

  context "premium agents in denver" do
    let(:search_location) { "Denver, CO" }

    let!(:premium_agent1) do
      create(
        :company_profile,
        :premium,
        :agent,
        lat: denver_city.lat,
        lng: denver_city.lng,
      )
    end
    let!(:premium_agent2) do
      create(
        :company_profile,
        :premium,
        :agent,
        lat: denver_city.lat,
        lng: denver_city.lng,
      )
    end

    it "returns just the featured agents" do
      expect(subject).to include(featured_premium_agent)
      expect(subject).to_not include(premium_agent)
    end

    # More tests
  end
end
```

## Good

```ruby
context "premium agents in denver" do
  it "returns just the featured agents" do
    search_location = "Denver, CO"
    featured_premium_agent = create(
      :company_profile,
      :premium,
      :agent,
      :featured,
      lat: denver_city.lat,
      lng: denver_city.lng,
    )
    premium_agent = create(
      :company_profile,
      :premium,
      :agent,
      lat: denver_city.lat,
      lng: denver_city.lng,
    )

    match_results = Agent::Match::Result.new(search_location: search_location).fetch

    expect(match_results).to include(featured_premium_agent)
    expect(match_results).to_not include(premium_agent)
  end

  # More tests
end
```

## Bad

```ruby
context "lorem ipsum" do
  let!(:record1) { create(:record1) }
  let!(:attribute1) { create(:attribute1) }
  subject { described_class.new.fetch(attribute1) }

  context "context1" do
    let(:record2) { create(:record2) }

    it "expects something" do
      expect(subject.count).to eq(1)
    end
  end

  context "context2" do
    let(:record3) { create(:record3) }
    let(:attribute1) { create(:attribute1, atr1: "a1") // here we are overriding attribute for this specific context

    it "expect something else" do
      expect(subject.count).to eq(2)
    end
  end
end
```

## Good

```ruby
context "lorem ipsum" do
  context "context1" do
    it "expects something" do
      create(:record1)
      create(:record2)
      attribute1 = create(:attribute1)

      data = described_class.new(attribute1).fetch

      expect(data.count).to eq(1)
    end
  end

  context "context2" do
    it "expect something else" do
      create(:record1)
      record3 = create(:record3)
      attribute1 = create(:attribute1, atr1: "a1")

      data = described_class.new(attribute1).fetch

      expect(subject.count).to eq(2)
    end
  end
end
```

## Further Reading
- https://thoughtbot.com/blog/lets-not
