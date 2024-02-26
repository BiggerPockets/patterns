---
categories: RSpec
name: Prefer request specs over controller specs
---

[Request specs](https://rspec.info/features/6-0/rspec-rails/request-specs/request-spec/) are preferred over [Controller specs](https://rspec.info/features/6-0/rspec-rails/controller-specs/).

Rspec's 3.5 release notes https://rspec.info/blog/2016/07/rspec-3-5-has-been-released/
> The official recommendation of the Rails team and the RSpec core team is to write request specs instead.
> Request specs allow you to focus on a single controller action, but unlike controller tests involve the router,
> the middleware stack, and both rack requests and responses. This adds realism to the test that you are writing,
> and helps avoid many of the issues that are common in controller specs.


### 👎 Controller specs

- Test a single HTTP request in each example.
- Only one controller acton is tested, the views are not loaded by default.
- Coupled with controller implementation. When refactoring you can have the external behavior of the application unchanged, but controller spec can fail.

### 👍 Request specs

- Can test multiple HTTP requests in each example.
- Tests full request response cycle, including routing, controllers, and views. Request spec behaviour is closer to the execution in production environment than controller spec.
- Less cupled with controler behaviour.

### 🏎️ Speed

Comparing the run times of a controller and request spec showed that the request spec is 20% faster than the controller spec.

#### Controller spec
```ruby
RSpec.describe Billing::CheckoutController do
  it "redirects to stripe checkout session url", :mock_stripe do
    user = create(:social_user)
    log_in_as user
    bootcamp_price_id = stripe_helper.create_price(id: "bootcamp_test_id").id
    subscription_price_id = stripe_helper.create_price(id: Billing::Plan.pro_annual.id).id

    purchase_bundle = create(:purchase_bundle, price_ids: [bootcamp_price_id, subscription_price_id])

    get :show, params: { id: purchase_bundle.slug }

    expect(response).to redirect_to("https://checkout.stripe.com/pay/test_cs_3")
  end
end
```


#### Request spec
```ruby
RSpec.describe "Billing Checkout", type: :request do
  it "redirects to stripe checkout session url", :mock_stripe do
    user = create(:social_user)
    log_in_as user
    bootcamp_price_id = stripe_helper.create_price(id: "bootcamp_test_id").id
    subscription_price_id = stripe_helper.create_price(id: Billing::Plan.pro_annual.id).id

    purchase_bundle = create(:purchase_bundle, price_ids: [bootcamp_price_id, subscription_price_id])

    get billing_checkout_path(id: purchase_bundle.slug)

    expect(response).to redirect_to("https://checkout.stripe.com/pay/test_cs_3")
  end
end

```

More documentation:
- https://medium.com/just-tech/rspec-controller-or-request-specs-d93ef563ef11
- https://stackoverflow.com/questions/40851705/controller-specs-vs-request-specs
