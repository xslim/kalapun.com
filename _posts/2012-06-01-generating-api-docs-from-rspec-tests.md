---
title: "Generating API docs from RSpec tests"
tags: [ruby,tests,rspec,api,doc]
---

In this small post I'll show how I use RSpec to test and generate docs for my API.

Full documentation for the gem and it's DSL you can find on it's homepage [rspec_api_documentation](http://github.com/zipmark/rspec_api_documentation)

First of all, you will need nice gem `rspec_api_documentation`, so edit your `Gemfile` and add

``` ruby
gem 'rspec_api_documentation', :group => [:test, :development], :git => "git://github.com/zipmark/rspec_api_documentation.git"

```

## Rake helper task
Before we proceed, Let's create rake task for testing API without generation `lib/tasks/spec_api.rake`

``` ruby
require 'rspec/core/rake_task'

desc 'Run API specs test in spec/acceptance'
  RSpec::Core::RakeTask.new('spec:api') do |t|
  t.pattern = 'spec/acceptance/**/*_spec.rb'
  #t.rspec_opts = ["--format RspecApiDocumentation::ApiFormatter"]
end
```


Now if we will run `rake spec:api` it will run tests in files `spec/acceptance/*_spec.rb` for testing API without generationg docs. So don't forget to create that folder.

## Spec helper & override
Also, I create `spec/api_helper.rb` that contains some helpers for my tests.

``` ruby
def prefill_db_for_api
  puts "Creating DB data"

  # Some factory_girl DB prefilling

end

# Overrides for customizing API docs generator

module RspecApiDocumentation
  class RackTestClient < ClientBase

    def request_headers
      my_headers = env_to_headers(last_request.env)
      my_headers.reject! {|k,v| v.empty?}
      my_headers.reject! {|k,v| (v == 'example.org')}
    end

    alias_method :old_do_request, :do_request
    def do_request(method, path, params)
      old_do_request(method, "#{$base_url}#{path}", params)
    end
  end

  class Curl
    def url
      "#{host}#{$base_url}#{path}"
    end
  end

end
```


## Spec API test Example
I'll show you a sample test file
`spec/acceptance/business_spec.rb`

``` ruby
require 'spec_helper'
require 'api_helper'
require 'rspec_api_documentation/dsl'

resource "Business v1.1" do

  before do
    prefill_db_for_api
  end

  $base_url = '/api/v1_1'
  let(:client) { RspecApiDocumentation::RackTestClient.new(self, :headers => { "HTTP_ACCEPT" => "application/json" }) }

  post "/login" do
    parameter :email, "Employee email", :required => true
    parameter :password, "Employee password", :required => true

    let(:email) { Employee.first.email }
    let(:password) { 'test22' }

    scope_parameters :employee, :all

    example "Authenticate with login and password" do
      do_request
      #puts 'login resp ' + response_body.inspect
      status.should == 200

      employee = Employee.first
      item = JSON.parse(response_body)
      item["authentication_token"].should_not be_empty
      item["authentication_token"].should eq(employee.authentication_token)
      item["email"] == employee.email
    end

  end

    get "/products" do

    # Explictly pre-set authentication_token
    let(:authentication_token) { Employee.first.authentication_token }
    let(:client) { RspecApiDocumentation::RackTestClient.new(self, :headers => { "HTTP_ACCEPT" => "application/json", 'HTTP_X_RABATME_AUTH_TOKEN' => authentication_token }) }

    example "Load products for current shop" do
      explanation "Products that are only available for this shop. Model: ShopProduct"

      do_request
      status.should == 200

      product = ShopProduct.first
      items = JSON.parse(response_body)
      puts items.inspect
      item = items.first
      item["id"].should_not be_empty
      item["id"].should eq(product._id.to_s)
      item["points"].should eq(product.points)
    end
  end

end
```


Now when you will run `rake docs:generate ` it will run tests & generate API docs for you. If you wont only to run tests - you do `rake spec:api`
