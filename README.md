# APIKits Ruby Client

**ApiKits** handles all the logic necessary to call a REST API, catch the response and initialize a parsed object. It works very well with Ruby on Rails, so you can exchange your ActiveRecord-based models without any concern and your application will make API calls transparently. It is possible to use Typhoeus for parallel requests, but it uses  the native Ruby Library Net::Http by default.

## Installation

Add this line to your application's `Gemfile`:
```ruby
gem 'apikits-ruby'
```

And then execute:
```
$ bundle
```

Or install it yourself as:
```
$ gem install apikits-ruby
```

If you will use Typhoeus (https://github.com/typhoeus/typhoeus), you must have Typhoeus version above 0.5.0.

## Documentation

You can find the code documentation for the last version generated by Yard on this link:

[http://rdoc.info/gems/apikits-ruby/frames](http://rdoc.info/gems/apikits-ruby/frames)

## Basic Usage

Create an initializer:
```ruby
ApiKits.configure do |config|
  # Define the API entry point
  config.path = 'http://api.example.com'
  # Default header
  config.header = { 'param1' => '123329845729384759237592348712876817234'}
  # JWT Bearer Token Authorization
  config.auth('243u2484290842048108-1ew-wf09834')
  # If inside Rails
  config.mock = Rails.env.test?
end
```

Add this to your `ApplicationController`:
```ruby
rescue_from ApiKits::Exceptions::NotFound, :with => :not_found

def not_found
  # Specify your own behavior here
end
```

You can define a more generic rescue that will work for any error:
```ruby
rescue_from ApiKits::Exceptions::Generic, :with => :generic_error
```

In your model, inherit from  `ApiKits::Base`:
```ruby
class User < ApiKits::Base
```

Then, in your action, call the regular methods on your model:
```ruby
User.all # or User.collection
User.find(3) # or User.get(3)
User.delete(3) # or User.destroy(3)
User.update_attributes({ name: 'Fred' }) # or User.put({ name: 'Fred' })
User.create({ name: 'Wilma' }) # or User.post({ name: 'Wilma' })
```
where 3 is the `id` of the user.

## Advanced Usage

ApiKits can read API responses with root nodes based on the name of the virtual class.
In some cases, that is not the required behavior. To redefine it, use `root_node` method:
```ruby
class Admin < ApiKits::Base
  self.root_node = 'user'
end
```

To specify a resource path different from the expected, you can overwrite the behavior by setting `resource_path`:
```ruby
class Admin < ApiKits::Base
  self.resource_path = 'users?type=admin'
end
```

ApiKits can handle associations. It will automatically instantiate an associations for you:
```ruby
class Person < ApiKits::Base
  self.associations = { :houses => "House", :cars => "Car" }
end

person = Person.first
person.houses
person.cars
```
This code will create a getter and a setter for houses and cars and initialize the respective classes.

When you are working with collections you can use API pagination in accordance with the  HAL Specification (http://stateless.co/hal_specification.html) .
You just need to pass a hash as:
```ruby
{
  total: 10,
  total_pages: 1,
  offset: 10,
  _links: {
    first: { href: "/api/clients" },
    previous: null,
    self: { href: "/api/clients" },
    next: null,
    last: { href: "/api/clients?page=1" }
  },
  users: [ { user: { name: "example1" } }, { user: { name: "example2" } } ]
}
```

## Parallelism

When making parallel requests, the requests are made in threads, so to get the response it is necessary to specify an initialized object to update when the request is complete:
```ruby
@users = ApiKits::Collection.new({}, User)
@cars  = ApiKits::Collection.new({}, Car)
@house = House.new

ApiKits.parallel do
  User.all.on_complete_update(@users)
  Car.all.on_complete_update(@cars)
  House.find(1).on_complete_update(@house)
end
```

## Testing

Set mock config variable as true inside your `spec_helper` or `test_helper`:
```ruby
ApiKits.configure do |config|
  config.mock = true
end
```

With this setting no requests will be made. All calls will just return a new object with the attributes received.

## Changelog

[https://github.com/apikits/apikits-ruby/blob/master/CHANGELOG.md](https://github.com/apikits/apikits-ruby/blob/master/CHANGELOG.md)

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Credit

I would like to thank [Paulo Henrique Lopes Ribeiro](https://github.com/plribeiro3000) for his [api-client](https://github.com/zertico/api-client) gem which provided a lot of the inspiration and framework for ApiKits.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## Copyright

Copyright © 2016 Jurgen Jocubeit. All rights reserved.