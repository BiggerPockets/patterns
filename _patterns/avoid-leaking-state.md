---
categories: RSpec
name: Avoid leaking state between tests
---

Let's about state leaking from one test to the next. One of the ways state may leak is by not disabling a
feature flag that was enabled for a test.

## Disable enabled feature flags

## Bad

````ruby
  before { Flipper.enable(:feature_flag) }

  it "passes when the feature flag is enabled" do
    expect(subject).to eq expected_value
  end
````

## Good

````ruby
  before { Flipper.enable(:feature_flag) }
  after { Flipper.disable(:feature_flag) }

  it "passes when the feature flag is enabled" do
    expect(subject).to eq expected_value
  end
````

````ruby
  it "passes when the feature flag is enabled" do
    Flipper.enable(:feature_flag)
    expect(subject).to eq expected_value
    Flipper.disable(:feature_flag)
  end
````

## Unfreeze time after it's frozen

## Bad

````ruby
  before { freeze_time }

  it "passes when the time is frozen" do
    expect(subject).to eq expected_value
  end
````

## Good

````ruby
  before { freeze_time }
  after { unfreeze_time }

  it "passes when the time is frozen" do
    expect(subject).to eq expected_value
  end
````

````ruby
  it "passes when the time is frozen" do
    freeze_time do
      expect(subject).to eq expected_value
    end
  end
````

## Travel back to the current time after time traveling

## Bad

````ruby
  before { travel 1.day }

  it "passes when the time travelling" do
    expect(subject).to eq expected_value
  end
````

````ruby
  before { travel_to Time.zone.local(2004, 11, 24, 1, 4, 44) }

  it "passes when the time travelling" do
    expect(subject).to eq expected_value
  end
````

## Good

````ruby
  before { travel 1.day }
  after { travel_back }

  it "passes when the time travelling" do
    expect(subject).to eq expected_value
  end
````

````ruby
  it "passes when the time travelling" do
    travel(1.day) do
      expect(subject).to eq expected_value
    end
  end
````

````ruby
  before { travel_to Time.zone.local(2004, 11, 24, 1, 4, 44) }
  after { travel_back }

  it "passes when the time travelling" do
    expect(subject).to eq expected_value
  end
````

````ruby
  it "passes when the time travelling" do
    travel_to(Time.zone.local(2004, 11, 24, 1, 4, 44)) do
      expect(subject).to eq expected_value
    end
  end
````
