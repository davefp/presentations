# Gems You Should Never Leave Development Without

### David Underwood

![](https://s3.amazonaws.com/ooomf-com-files/KJyFV5SZSweiYGhMhrqC_MD4817.jpg)

---

# Before We Start

---

# Development Gems!

---

# quiet_assets

Makes assets shut the hell up in development

```
Started GET "/assets/application.css?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery_ujs.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery.ui.mouse.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery.ui.sortable.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery.ui.widget.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/jquery.ui.core.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/turbolinks.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/bootstrap/tab.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/bootstrap/alert.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/cocoon.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400

Started GET "/assets/tinymce/preinit.js?body=1" for 127.0.0.1 at 2014-07-21 22:38:53 -0400
```

---

# better_errors

Adds a much better error page for development.

Also a REPL if you add `binding_of_caller` too!

---

![original](https://camo.githubusercontent.com/0287f64b56055d81a05f57a50400bc264614476d/687474703a2f2f692e696d6775722e636f6d2f367a42474141622e706e67)

---

# pry + pry-debugger

Pry is a great alternative to IRB.

Pry-debugger adds your favourite debug commands:

* step
* next
* continue
* finish

Allows you to set breakpoints too.

---

# OK, Let's Talk Production

^ Your development environment probably has some of the following characteristics: Single user, local to your machine, secure

^ Production is rather different: It's multi-user, completely remote, and runs over an untrusted network (i.e. the internet)

---

# Environment Management

![](https://s3.amazonaws.com/ooomf-com-files/LGhxuAbT5Wop4JYcrMpV_IMG_3808.jpg)

---

# figaro

```ruby
# Gemfile
gem 'figaro'

# config/application.yml
pusher_app_id: "2954"
pusher_key: "7381a978f7dd7f9a1117"
pusher_secret: "abdc3b896a0ffb85d373"

production:
  pusher_app_id: "5112"
  pusher_key: "ad69caf9a44dcac1fb28"
  pusher_secret: "83ca7aa160fedaf3b350"

# Anywhere
Figaro.env.pusher_key
```

^ Figaro is Rails-only
^ Has a rake task to copy config to Heroku

---

# dotenv

```ruby
# Gemfile
gem 'dotenv'

# config.ru
require 'dotenv'
Dotenv.load

# .env
PUSHER_APP_ID=2954
PUSHER_KEY=7381a978f7dd7f9a1117
PUSHER_SECRET=abdc3b896a0ffb85d373

# Anywhere
ENV['PUSHER_KEY']
```

^ Works with anything
^ Not as many bells and whistles as figaro: no environment support, not Heroku push, etc.

---

# Error Reporting

![](http://666a658c624a3c03a6b2-25cda059d975d2f318c03e90bcf17c40.r92.cf1.rackcdn.com/unsplash_526360a842e20_1.JPG)

---

# exception_notification

```ruby
# Gemfile
gem 'exception_notification'

# config/environments/production.rb
Whatever::Application.config.middleware.use ExceptionNotification::Rack,
  email: {
    email_prefix: "[Oh Shiii] ",
    sender_address: %{"notifier" <notifier@example.com>},
    exception_recipients: %w{exceptions@example.com}
  }
```

^ Email notification is dead simple. If your app can send email you can use this
^ Lacks the bells and whistles of full-blown services

---

# Exception Notification Services

* Airbrake
* Errbit
* Honeybadger
* Rollbar
* Lots more

Ruby Toolbox has a great list: [https://www.ruby-toolbox.com/categories/exception_notification](https://www.ruby-toolbox.com/categories/exception_notification)

^ Neat things you can do with these services: Group similar exceptions. Create GH issues. Link back to source code. Multiple users.
^ Downside is they start to cost money or require self-hosting (Errbit).

---

# Application Server

![](https://s3.amazonaws.com/ooomf-com-files/3CoEETpvQYO8x60lnZSA_rue.jpg)

---

# What's Available?

* WEBrick
* Thin
* Unicorn
* Passenger
* Puma

Go with Unicorn unless you have a reason not to

Don't use WEBrick in production

^ WEBrick is the Rails default, and is part of the standard library. That's about all there is to say about it
^ Thin is an evented app server built on event_machine.
^ Unicorn is multi-threaded
^ Passenger is a 'total' solution (no need for a separate routing web server)
^ Puma is fast and concurrent

---

# Anything Else?

# How About... Middleware!

![](https://s3.amazonaws.com/ooomf-com-files/OSASuBX1SGu4kb3ozvne_IMG_1088.jpg)

---

# rack-canonical-host

Redirect all requests to a specific host. Can also be used to force SSL. Great for custom domains on Heroku

```ruby
# Gemfile
gem 'rack-canonical-host'

# config.ru (Rails example)
require ::File.expand_path('../config/environment',  __FILE__)

use Rack::CanonicalHost, ENV['CANONICAL_HOST'], force_ssl: true
run YourRailsApp::Application
```

^ supports tons of options including regex matching on host strings

---

# rack-attack

Connection whitelists, blacklists, and throttling. Stops misbehaving clients from even hitting your app code

```ruby
# Gemfile
gem 'rack-attack'

# config/application.rb
config.middleware.use Rack::Attack

# config/initializers/rack-attack.rb
class Rack::Attack
  # your custom configuration...
end
```

^ Works well with Fail2Ban blacklists and others

---

# rack-google-analytics

Adds an analytics code to every page of your app

```ruby
# Gemfile
gem 'rack-google-analytics'

# config.ru
use Rack::GoogleAnalytics, tracker: 'UA-xxxxxx-x'
```

---

# Heroku

![](https://s3.amazonaws.com/ooomf-com-files/bXoAlw8gT66vBo1wcFoO_IMG_9181.jpg)

---

# rails_12factor

Tells Rails to serve assets. This is usually delegated to the routing webserver (e.g. nginx or apache)

Redirects logs to stdout rather than logfiles

Stops Heroku complaining every time you deploy

```ruby
# Gemfile
ruby '2.0'
gem 'rails_12factor'
```

---

# Personal Favourites?

---

# [fit] Thanks!