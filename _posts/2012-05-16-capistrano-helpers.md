---
title: "Capistrano helpers"
tags: [ruby, rails, capistrano]
---

While working on new rails project, I've made few lazy helpers for Capistrano `cap` command that I want to share. All of them are in my `deploy.rb` file.



Forwarding your id_rsa key for remote pulling the repo

``` ruby
ssh_options[:forward_agent] = true
before 'deploy:update', 'local:sshadd'

namespace :local do
  desc "Localy run ssh-add"
  task :sshadd, :roles => :app do
    system "ssh-add"
  end
end
```


Running remote bundle install

``` ruby
after 'deploy:update', 'bundle:install'

namespace :bundle do
  desc "Installs the application dependencies"
  task :install, :roles => :app do
    # For production, exclude gems from development & test groups
    run "cd #{current_path} && #{bundle_cmd} --without development test"
  end
end
```


Restarting Thin server

``` ruby
after 'deploy:update', 'thin:restart'

namespace :thin do
  desc "Restart Thin"
  task :restart do
    invoke_command "cd #{current_path} && bundle exec thin restart -C #{deploy_to}/current/thin.yml"
  end
end
```


Helpers to syncing local and remote MongoDB

``` ruby
set :mongodbname_prod, 'ximity_me_development'
set :mongodbname_dev, 'ximity_me_development'

namespace :syncdb do

  desc 'Synchronize MongoDB local -> production.'
  task :dev2prod, :hosts => "#{application}" do
    database = mongodbname_dev
    dev_database = database
    filename = "database.#{database}.#{Time.now.strftime '%Y-%m-%d_%H-%M-%S'}.tar.bz2"
    system "/usr/local/bin/mongodump -db #{database}"
    system "tar -cjf #{filename} dump/#{database}"
    upload filename, "#{shared_path}/#{filename}"
    system "rm -rf #{filename} | rm -rf dump"

    database = mongodbname_prod
    run "tar -xjvf #{shared_path}/#{filename}"
    run "/usr/local/bin/mongorestore #{fetch(:db_drop, '')} -db #{database} dump/#{dev_database}"
    run "rm -rf dump"
  end

  desc 'Synchronize MongoDB production -> local.'
  task :prod2dev, :hosts => "#{application}" do
    database = mongodbname_dev
    dev_database = database
    filename = "database.#{database}.#{Time.now.strftime '%Y-%m-%d_%H-%M-%S'}.tar.bz2"
    run "cd #{shared_path} && /usr/local/bin/mongodump -db #{database}"
    run "cd #{shared_path} && tar -cjf #{filename} dump/#{database}"
    download "#{shared_path}/#{filename}", "#{filename}"
    run "cd #{shared_path} && rm -rf #{filename} | rm -rf dump"

    database = mongodbname_prod
    system "tar -xjvf #{filename}"
    system "/usr/local/bin/mongorestore #{fetch(:db_drop, '')} -db #{database} dump/#{dev_database}"
    system "rm -rf #{filename}"
    system "rm -rf dump"
  end

  desc 'Generate indexes.'
  task :reindex, :host => "#{application}" do
    run "cd #{current_path} && bundle exec rake db:mongoid:create_indexes --trace"
  end

end
```


Do you have any useful helpers for `cap` ?
