---
categories: Rails
name: Use namespacing for apps and events
---

We deploy one app to Heroku. But it's a huge app and there are smaller apps within the larger app.

Some examples of these apps:

* Rent estimator
* Search
* Agent finder
* Rehab estimator
* Forum
* Blog

When evolving cross cutting concerns, let's start thinking in terms of these separate apps and namespace them.

## Problems with one big app

* Blast radius for changes can affect many apps at once
* No guidelines on namespaces so codebase can be difficult to navigate
* When we come to splitting up the app, it'll be difficult to know where to start and it'll get harder as we glom more code into our monolithic codebase
* Can devolve into a Ball Of Mud

## What is namespacing?

Describe an app by using period separated namespaces.

Examples from the above:

* Rent estimator - `calculators.rent_estimator`
* Search - `tools.search`
* Agent finder - `marketplace.agent_finder`
* Rehab estimator - `calculators.rehab_estimator`
* Forum - `community.forum`
* Blog - `content.blog`

## Why namespace?

* Communicates clearly which app an area of code is for
* Allows for namespacing controllers, models eg: `Marketplace::AgentFinder::Agent`
* Allows targeting all tools, all platform apps using wildcards like `tools.*`

## Structure of an app namespace

`<area>.<app_name>`

## Structure of an event namespace

`<app_namespace>.<domain_object>.<verb_in_past_tense>`

* Do not put technology terms in the namespace such as `stripe`, `iterable` etc
* Instead, add the domain_object such as `invoice`, `customer`, `email` or `contact`

**Good event names**

* `billing.invoice.paid`
* `marketing.campaigns.email.sent`

**Poor event names**

* `stripe.invoice.paid` - do not put the technology in the event name. This is an implementation detail and can be put into the payload.
* `marketing.campaigns.email_send` - no domain object, last section in present tense
* `contact.updated` - no namespace, no idea which area of the code this affects

## Examples

* [Events](https://github.com/BiggerPockets/biggerpockets/blob/dffe76a35df84b26ca7f30e5061d0202348e8d66/lib/events/event.rb#L45-L49)
* [Features](https://github.com/BiggerPockets/biggerpockets/pull/16341/files#diff-55fe8f75e59662410da141d170d53137b121237ca215d6cbd8465278cb97a66dR10-R12)