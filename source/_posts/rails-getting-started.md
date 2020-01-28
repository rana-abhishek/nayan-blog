---
title: Getting Started with Ruby on Rails
date: 2020-01-28 08:00:00
author: Anuj Middha
category: Rails
tags:
- Ruby
- Rails
---

{% asset_img ruby-on-rails.png %}

We recently inducted a couple of engineers into our Rails team. Both of them had a web frontend experience, but zero experience with Ruby or backend development. Thanks to the simplicity and convention focussed approach of the Rails framework, both of them were writing test driven production grade code within a week!

First thing was setting up their machines. We decided to go with

- Ubuntu 18.04 for the OS
- RVM as the Ruby version manager
- RubyMine as the IDE
- Postgresql as the database

### Ruby
First step was to get them comfortable with Ruby. We believe that doing is the best way of learning. So we got them to complete the wonderful koans at http://rubykoans.com/ to get hands on practice.

An important thing while working in a team is to have consistent coding style across all members. We follow the style guide at https://github.com/rubocop-hq/ruby-style-guide , so they read through the guide.

### Rails
We feel the best guide for Rails is the official guide itself.

https://edgeguides.rubyonrails.org/

The three sections that were assigned were,

- Getting Started with Rails
- Models
- Controllers

We skipped the views as we mostly work on API only apps.

After this, we covered the Rails style guide https://github.com/rubocop-hq/rails-style-guide .

### Tests
We use [RSpec](https://github.com/rspec/rspec-rails) and [FactoryBot](https://github.com/thoughtbot/factory_bot) internally for writing our tests.

For RSpec, the Github page is a good starting point https://github.com/rspec/rspec-rails

For FactoryBot, we assigned the Getting Started guide on Github https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md

### Continuing Education
With these basic tutorials, the engineers were basically ready for contributing to production. Their first few pull requests had many comments, but they came down significantly within the first two weeks.

For continuing our Rails education, we keep reading up on the frequent gems that we use, such as

- devise
- aasm
- active_model_serializers
- pundit
- resque
- whenever
- carrierwave

and other excellent gems.

### Conclusion
We got a pleasant reminder as to why Rails is our favorite framework to work on. Within the first few weeks only the engineers were writing production grade, well tested code.

Hats off to the Ruby philosophy and Matz that our fresh Ruby engineers were able to start guessing the function names for different classes almost immediately!
