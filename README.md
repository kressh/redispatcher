![Redispatcher](https://zhf.io/p8IRct/raw)

[![Gem Version](https://badge.fury.io/rb/redispatcher.svg)](http://badge.fury.io/rb/redispatcher)
[![Code Climate](https://codeclimate.com/github/rambler-digital-solutions/redispatcher/badges/gpa.svg)](https://codeclimate.com/github/rambler-digital-solutions/redispatcher)
[![Test Coverage](https://codeclimate.com/github/rambler-digital-solutions/redispatcher/badges/coverage.svg)](https://codeclimate.com/github/rambler-digital-solutions/redispatcher)
[![Build Status](https://travis-ci.org/rambler-digital-solutions/redispatcher.svg?branch=master)](https://travis-ci.org/rambler-digital-solutions/redispatcher)

This gem provides delivery of objects from database to any structure with power of callbacks.

Supporting ORM are ActiveRecord and Sequel (through [sequel-rails](https://github.com/TalentBox/sequel-rails) gem)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'redispatcher'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install redispatcher

## Using

### Writing Dispatchers

Dispatchers inherit from `Redispatcher::Dispatcher`, live in your `app/dispatchers` directory, and are named for the model that they dispatch.

You can use rails generator to generate dispatcher:

```
rails g dispatcher Topic
```

This will create TopicDispatcher:

```ruby
# app/dispatchers/topic_dispatcher.rb
class TopicDispatcher < Redispatcher::Dispatcher
  # ...
end
```

Do not hesitate to use dispatscher's callbacks `before_initialize`, `after_initialize`, `before_process`, `after_process`, `before_commit`, `after_commit`, `before_rollback`, `after_rollback` just like that:

```ruby
# app/dispatchers/topic_dispatcher.rb
class TopicDispatcher < Redispatcher::Dispatcher

  after_initialize do
    @processed_object = {}
  end

  before_process do
    @processed_object.merge! title: object.title
  end

  after_commit :update_mongodb

  def update_mongodb
    MONGO['topics'].update({ id: object.id }, processed_object, upsert: true)
  end
end
```

### Enable dispatcher for your model

With ActiveRecord:

```ruby
class Topic < ActiveRecord::Base
  dispatchable
end
```

With Sequel:

```ruby
class Topic < Sequel::Model
  dispatchable
end
```

### Dispatch!

Just call dispatch method on object you going to dispatch.

```ruby
dispatched_topic = Topic.first.dispatch
```

## License

Redispatcher is licensed under the MIT license. See LICENSE for full license text.

## Contributing

1. Fork it ( https://github.com/rambler-digital-solutions/redispatcher/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
