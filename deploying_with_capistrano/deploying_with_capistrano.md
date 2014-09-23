footer: David Underwood @davefp

# Deploying Your Apps (With Capistrano)

---

# [fit] What is 'Deployment'?

---

# Deployment is the process of *distributing* and *starting* your app in an *environment*

---

# [fit] What Steps do we Need?

---

# What do you do in development?

---

# One-Time Operations

* Provision server
* Set up database and other supporting services
* Create credentials for external services

^ Capistrano (and this talk) don't cover these tasks.

---

# Per-Deploy Operations

* Fetch new code
* Run migrations
* Update configuration
* Restart (or stop/start) app

^ These are the tasks that Capistrano can help you with.

---

# [fit] Capistrano

---

# The Basics

Add it to your gemfile

```ruby
gem 'capistrano'
```

Get it installed

```
bundle install
bundle exec cap install
```

^ Pretty basic stuff so far

---

# Anatomy of a Command

Specify a **stage**, and a **task** to run on it:

```bash
cap production deploy

cap staging deploy:migrate

cap custom_stage custom_namespace:custom_task
```

^ We'll come back to exactly what this does in a minute

---

# Configuration

Your `Capfile` contains any includes you might need.

```ruby
require 'capistrano/setup'

require 'capistrano/deploy'

require 'capistrano/rbenv'

require 'capistrano/rails'

require 'capistrano3/unicorn'

Dir.glob('lib/capistrano/tasks/*.cap').each { |r| import r }
```

^ Common tasks (e.g. deploying Rails apps) are covered by pre-existing gems.

^ Here's a Capfile from a project of mine. You can see I'm including all the default Cap tasks at the bottom, and also pulling in specific tasks from some gems (rbenv, rails, unicorn)

---

Configuration common to all states is found in `config/deploy.rb`

```ruby
lock '3.2.1'

set :application, 'arenagym'
set :repo_url, 'git@bitbucket.org:davefp/arenagym.git'

set :linked_files, %w{config/database.yml .env}
set :linked_dirs, %w{tmp/pids}

set :unicorn_config_path, "config/unicorn.rb"
set :unicorn_rails, true

set :rbenv_type, :user
set :rbenv_ruby, '2.1.1'
set :rbenv_map_bins, %w{rake gem bundle ruby rails}
set :rbenv_roles, :all # default value
```

^ Capistrano makes heavy use of a declarative dsl here. It barely looks like ruby!

---

Environment/stage specific config in named files, e.g. `config/deploy/production.rb`

```ruby
set :stage, :production

set :deploy_to, '~/apps/arenagym'

set :branch, 'master'

set :rails_env, 'production'

# Simple Role Syntax
# ==================
# Supports bulk-adding hosts to roles, the primary
# server in each group is considered to be the first
# unless any hosts have the primary property set.
role :app, %w{deploy@arenagym.net:2222}
role :web, %w{deploy@arenagym.net:2222}
role :db,  %w{deploy@arenagym.net:2222}
```

---

# Take a look at the deploy task:

## [Handy Link](https://github.com/capistrano/capistrano/blob/master/lib/capistrano/tasks/framework.rake)

^ Notice that this doesn't do anything with regard to invoking your app or other services. By default, Capistrano is language and framework agnostic

---

# Linked Files

Linked files are any files that are not part of your codebase that you need to run your app.

They persist from deploy to deploy

* database.yml
* .env, config.yml, or secrets.yml
* pid files (e.g. for unicorn restarts)

---

# Deploys

Each time you deploy, your code is put in a new folder alongside the old stuff.

When everything is ready, this folder is symlinked to the 'current' folder to minimise downtime.

A certain number of previous releases are kept around so that you can roll back.

---

# Tailoring Capistrano to your needs

* **[capistrano/rails](https://github.com/capistrano/rails)** provides migration and asset tasks.

* **[capistrano3/unicorn](https://github.com/tablexi/capistrano3-unicorn)** provides app server start, stop, restart tasks.

* Etc..

---

# [fit] Questions?

---

# Thanks!

Capistrano site: [capistranorb.com](http://capistranorb.com/)

This presentation: [On my GitHub](https://github.com/davefp/presentations/blob/master/deploying_with_capistrano/deploying_with_capistrano.md)

My blog: [theflyingdeveloper.com](http://theflyingdeveloper.com)

