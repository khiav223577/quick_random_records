## QuickRandomRecords

`quick_random_records` is a Ruby Gem that empowers ActiveRecord Model to return random records dramatically fast, even with tables that have lots of data rows.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'quick_random_records'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install quick_random_records

## Usage

```ruby
# return ActiveRecord::Relation contains 10 random model objects from User Table

users = User.random_records(10)
```

## Dramatically fast, compared to common random records strategies

Scenario: query 100 random records from table with 550,000 data rows in localhost.

1. `quick_random_records` costs only `6.2ms` totally.
![alt text](https://user-images.githubusercontent.com/19776127/40586675-137b0f5a-61f8-11e8-85e3-4df7a96ed343.png)
2. `Model.order("RAND()").limit(num)` costs `3314.1ms`.
![alt text](https://user-images.githubusercontent.com/19776127/40585124-b6f7b0a2-61df-11e8-9884-86f96354efbc.png)
3. `Model.where(id: Model.pluck(:id).sample(num))` costs `1659.4ms` totally.
![alt text](https://user-images.githubusercontent.com/19776127/40585123-b6d07f00-61df-11e8-9622-e4cd61100e37.png)


![alt text](https://user-images.githubusercontent.com/19776127/40586737-e07cb99a-61f8-11e8-8d02-2a3dd4a832b5.png)

![alt text](https://user-images.githubusercontent.com/19776127/40585161-5add98b2-61e0-11e8-9265-11bef7a1536d.png)

## Fine-tuning

This strategy is fast because:

(1) Instead of plucking all id in the table, it selects id bewteen min_id and max_id.
    Then make complements if any missing records (id between min_id and max_id, but not exist in db). 

(2) It selects records 1.05 times more than required. So that it doesn't need to perform another query to make complements.
    And of course, it will truncate to required number of records before method return.
    
   You can configure your own multiply, which is 1.05 by default.
   EX: If table has 10% deleted records, multiply 1.1 will maximize the speed of random_records. 
    
 ```ruby
 # select 1.1 times more than required, that is 110 in this case. 
 # And it will truncate to 100 before method return.
    
 users = User.random_records(100, multiply: 1.1) 
 ```
   
## Drawback

This strategy works extremely well with table that has a lot of records and few deleted records.

But for tables with a lot of deleted records (ex: 8 deleted records out of 10 records),
it may return fewer random records as required since I limit the loop searching for complements.

The default `loop_limit` is `3`. You can configure your own `loop_limit` for searching complements.
```ruby
users = User.random_records(100, loop_limit: 5)
```

OR

You can use other strategy for tables with a lot of deleted records.

`Model.order("RAND()").limit(num)` is strategy 2 supported by this GEM
```ruby
users = User.random_records(100, strategy: 2)
```

`Model.where(id: Model.pluck(:id).sample(num))` is strategy 3 supported by this GEM
```ruby
users = User.random_records(100, strategy: 3)
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/quick_random_records. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
