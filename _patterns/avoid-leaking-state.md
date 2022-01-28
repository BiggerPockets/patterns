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
