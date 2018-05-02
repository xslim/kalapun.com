---
title: "Rails - SQLite and PostgreSQL playing nice together"
tags: [ruby, rails, sqlite, postgresql]
---

So I was asked to add few features to the web-app, that was in production and running PostgreSQL. Well, I didn't want to install the PostgreSQL on my local machine. So I decided to have development version and test version to run in SQLite, while keeping production version unchanged.

So I came up to few hacks.

## database.yml

First, edit your `config/database.yml` and change it so `development` and `test` enviroment will use SQLite

``` yaml
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
test: &test
  adapter: sqlite3
  database: db/test.sqlite3
  pool: 5
  timeout: 5000
```


And don't forget to run `rake db:reset` and `rake db:seed` to set your new DB and seed it with data.



## Gemfile

Don't forget to edit your `Gemfile`

``` ruby
group :development, :test do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
end
```


## SQL stuff

PostgreSQL has some SQL addons, that SQLite doesn't. Some of them can be changed to DB-agnostic style, and some of them - can be skipped for development.

As the first example, we have a query like

``` ruby
Thing.where("#{key} ilike ?", "%#{value}%")
```


You see this `iLike` keyword, don't you? Well the thing is, putting DB-agnostic query is even faster than native `ILIKE`. So change it to `lower(x) like lower(y)`:

``` ruby
Thing.where("lower(#{resource_class.table_name}.#{key}) LIKE lower(?)", "%#{value}%")
```


Next one is `INDEX`. PostgreSQL has an addition with `WHERE`, but SQLite doesnt. But for development purposes, we don't care.

``` ruby
  if Rails.env.development? || Rails.env.test?
    execute "CREATE UNIQUE INDEX websites_qa_profile_url ON websites (qa_profile_id, url)"
  else
    execute "CREATE UNIQUE INDEX websites_qa_profile_url ON websites (qa_profile_id, url) WHERE deleted_at IS NULL"
  end
```


This will use the proper SQL query depending on enviroment it runs.

## Dates

Next thing, PostgreSQL has his own dating formats like `now() - '5 days'::interval`, which SQLite doesn't have. So I've made this class

``` ruby
class AdapterSpecific

  class << self
    # Workaround for "updated_at > now() - '5 days'::interval"
    def datetime(timestring, modifier)
      res = ''
      case ActiveRecord::Base.connection.adapter_name
      when 'PostgreSQL'
        timestring = 'now()' if timestring == 'now'
        if modifier[0] == '-'
          add = '-'
          modifier[0] == ''
          modifier.strip!
        else
          add = '+'
        end
        "#{timestring} #{add} '#{modifier}'::interval"
      else
        "datetime(#{timestring}, #{modifier})"
      end
    end

    def before(modifier)
      case ActiveRecord::Base.connection.adapter_name
      when 'PostgreSQL'
        "now() - '#{modifier}'::interval"
      else
        "datetime('now', '-#{modifier}')"
      end
    end

  end
end
```


Put it as `lib/adapter_specific.rb`, edit your `config/initializers/application.rb` and add on top `require 'adapter_specific'`. Then change your SQL queries to be like one below:

``` ruby
Thing.where("updated_at > #{AdapterSpecific.before('5 days')}").limit(20)
```


Hope this short post will help you in playing with RoR apps.`
