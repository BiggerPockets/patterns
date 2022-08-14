---
categories: Practices
name: Cross-Team Channels for Wide-Ranging Changes
---

# Cross-Team Channels for Wide-Ranging Changes

When working on something that has broad engineering impact (see examples below), create a dedicated cross-team Slack channel and invite other engineers to join. 

## Why?

BiggerPockets has multiple engineeing teams, but the core application is a shared Rails monolith. We should solicit feedback from engineers from all teams when discussing or making wide-ranging changes. This also allows engineers who are _not_ interested in a topic to avoid involvement in discussions that they are not concerned about. And it helps keep the engineering channel dedicated to strategic, rather than tactical, discusssions.

## Tips

- Announce the new channel in the engineering channel.
- In general, keep the channel public so that any interested engineer can join in the discussion.
- Summarize long threads so that engineers in other timezones don't need to read the entire thread to find any decisions made.
- Couple the channel with a Notion document, especially if there are any architectural changes.

## Examples

Here are some topics that would make good cross-team channels:

- A major Rails, Ruby or database upgrade
- Dashboard redesign
- Migrating from one framework to another
- Implementing a new 3rd party service such as DataDog
- Logging, tracking, or security patterns

Examples of topics that are not good cross-team channels:

- A specific product feature (this should be discussed by the product team).
