# Iteration 0 - WebApp, CI/CD

> As a developer

> I want a web app setup with CI/CD

> So that I have delivery to the web

The aim of iteration 0 is to have all the code and tooling in
place to be able to regularily and confidently deploy to
production. These means setting up:

- The code frameworks - Rails and ReactJS
- Server configuration for servers, databases, etc
- a basic test suite
- A CI server to run the tests
- A CD script to push code to production
- Production or some kind of hosting.

In our example we will use Rails as the framework for the backend
and ReactJS as the Single Page App (SPA) frontend. We will use
RSpec as the main integration and rails testing framework and
Jest for frontend tests. This will be configured to work with a
free tier [Travis CI | Build Kite | Semaphore CI | other].
Finally we will deploy to a free tier Heroku app.

If there are any issues configuring this there will also be the
ability to use vagrant to use a pre configured machine setup.

Options for using the latest and greatest - as of the time of writing

- ruby 2.6.0 - https://www.ruby-lang.org/en/ 25 Dec 2018
  ```
  brew install rbenv
  rbenv install 2.6.0
  ruby -v
  ```
- node 11.8.0 - https://nodejs.org/en/ 24 Jan 2019
  ```
  brew install nvm
  nvm install 11.8.0
  node -v
    v11.8.0
  ```
- rails 6.0.0.beta1 https://rubygems.org/gems/rails/versions 18 Jan 2019
  ```
  gem install bundler
  gem install rails --pre
  rails -v
    Rails 6.0.0.beta1
  ```
- postgres
  TODO confirm? create default DB named username? some basic commands?
  ```
  brew install postgres
  brew services start postgres
  psql
  ```

## Create new Rails project

Create new Rails 6.0.0.beta1 project, using postgres as data base and RSpec as
testing framework.

TODO - how to make npm default over yarn?

```
rails new game-app --database=postgresql --skip-test

rails server # start dev rails server
rails s      # shorter syntax
open http://localhost:3000 # failure: FATAL: database "game_app_development" does not exist
CTRL-C       # kill server

rails db:create # to create DB
rails s         # restart the server
open http://localhost:3000
  Yay! You’re on Rails!
  Welcome
  Rails version: 6.0.0.beta1
  Ruby version: ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]
```

COMMIT ":tada: rails new"

Add RSpec with capybara, selenium webdriver, and screen shots.

```
# in Gemfile under group :development, :test add
group :development, :test do
  ...

  gem 'rspec-rails'
  gem 'rspec-example_steps'
  gem 'capybara'
  gem 'capybara-screenshot'
  gem 'selenium-webdriver'
  gem 'rspec-wait'
end

bundle install
bundle          # by default does the install

rails generate rspec:install  # to initialize rspec
rspec
  No examples found.

  Finished in 0.00031 seconds (files took 0.12398 seconds to load)
  0 examples, 0 failures

mkdir -p spec/features/flows/

cat > spec/features/flows/game-app_spec.rb
require 'rails_helper'

feature 'Game App', js: true do
  scenario 'I have a game app' do
    When 'user visits the app' do
      visit('/')
    end

    Then 'user sees they are on rails' do
      wait_for { page }.to have_content("Yay I'm on Rails")
      wait_for { page }.to have_content("Rails version: 6.0.0")
      wait_for { page }.to have_content("Ruby version: ruby 2.6.0")
    end
  end
end
^D

rspec
# Failure
  Failure/Error: raise ActionController::RoutingError, "No route matches [#{env['REQUEST_METHOD']}] #{env['PATH_INFO'].inspect}"

  ActionController::RoutingError:
    No route matches [GET] "/"

# add the default rails/welcome#index
vi config/routes.rb

Rails.application.routes.draw do
  ...
  get '/' => "rails/welcome#index"
end

# Failure
Failure/Error: wait_for { page }.to have_content("Yay I'm on Rails")
  expected to find text "Yay I'm on Rails" in "Yay! You’re on Rails!\nRails version: 6.0.0.beta1\nRuby version: ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"

Assuming we have no idea but we want to move on, we can pend the spec. Just say
that it is in progress, in fact it should actually fail.

> spec/features/flows/game-app_spec.rb
```

...
Then 'user sees they are on rails' do
pending "for some reason I am not on rails?"
wait_for { page }.to have_content("Yay I'm on Rails")
end

```

returns with the pending

```

    When user visits the app
    Then user sees they are on rails (FAILED)

I have a game app (PENDING: for some reason I am not on rails?)

```

COMMIT "a pending spec"

TODO currently COMMIT [:construction| specs setup and a pending spec](https://github.com/failure-driven/game-app/commit/ebde6532f3d4955d6c08de4351c72cccc65c4f25)

- split out the setup from the pending spec

# configure the browser and screen shots
vi spec/rails_helper.rb

    7 require 'rspec/rails'
    8 # Add additional requires below this line. Rails is not loaded until this point!
    9
   10 Dir['spec/support/**/*.rb'].each do |file|
   11   require Rails.root.join(file).to_s
   12 end

mkdir -p spec/support
mkdir -p tmp/capybara

cat > spec/support/capybara.rb
  require 'capybara/rspec'
  require 'capybara/rails'
  require 'capybara-screenshot/rspec'

  Capybara.register_driver :selenium_chrome do |app|
    Capybara::Selenium::Driver.new(
        app,
        browser: :chrome,
        desired_capabilities: {
            chromeOptions: {
                args: ['window-size=800,960']
            }
        }
    )
  end

  Capybara.javascript_driver = :selenium_chrome

  Capybara::Screenshot.webkit_options = {
    width: 1024,
    height: 768
  }

  Capybara::Screenshot.autosave_on_failure = false
  Capybara::Screenshot.prune_strategy = :keep_last_run

  RSpec.configure do |config|
    config.after do |example|
      if (example.metadata[:type] == :feature) && example.metadata[:js] && example.exception.present?
        Capybara::Screenshot.screenshot_and_open_image
      end
    end
  end
```

from the screen shot we see it is that "You're" on rails so updating our spec

> spec/features/flows/game-app_spec.rb

```{rb}
require 'rails_helper'

feature 'Game App', js: true do
  scenario 'I have a game app' do
    When 'user visits the app' do
      visit('/')
    end

    Then 'user sees they are on rails' do
      # note the apostrophe next line
      wait_for { page }.to have_content("Yay! You’re on Rails!")
      wait_for { page }.to have_content("Rails version: 6.0.0")
      wait_for { page }.to have_content("Ruby version: ruby 2.6.0")
    end
  end
end
```

passing test

```
    When user visits the app
    Then user sees they are on rails
  I have a game app

Finished in 1.6 seconds (files took 2.89 seconds to load)
1 example, 0 failures
```

COMMIT "passing spec for being on rails"

TODO again COMMIT [passing speec for being on rails](https://github.com/failure-driven/game-app/commit/2f5c9be4f02be0c5b49f44397cb3845cdf3a7039)

- split out capybara setup from the actual commit

```
Game App
Capybara starting Puma...
* Version 3.12.0 , codename: Llamas in Pajamas
* Min threads: 0, max threads: 4
* Listening on tcp://127.0.0.1:65404
    When user visits the app
    Then user sees they are on rails
  I have a game app

Finished in 1.84 seconds (files took 3.84 seconds to load)
1 example, 0 failures
```

But we want to confirm the rails and ruby version as well. For that we are
going to stop test exection with binding pry and play around with reading the
values on the page.
no binding pry

> Gemfile

```
group :development, :test do
  ...
  gem 'pry-rails'
  gem 'pry-stack_explorer'
  gem 'pry-byebug'
  gem 'better_errors', '2.1.1'
  ...
```

now `bundle install` or simply `bundle`

As the setup for `binding.pry` is now complete we can commit

COMMIT [add binding.pry and better tooling for debugging](https://github.com/failure-driven/game-app/commit/f9c7c5218ebc4959b98e1c8e21bc4481e8ab8399)

Having looked at the source of the default rails page we see that the version can be found with the CSS selector `p.version` but not being confident with how to pull out all the information we with Capybara we will put in a `binding.pry` to "pry" into the code and get a better idea.

> spec/features/flows/game-app_spec.rb

```{rb}
require 'rails_helper'

feature 'Game App', js: true do
  scenario 'I have a game app' do
    When 'user visits the app' do
      visit('/')
    end

    Then 'user sees they are on rails' do
      wait_for { page }.to have_content("Yay! You’re on Rails!")
      version = page.find("p.version")
      binding.pry
    end
  end
end
```

running this with `rspec` we are put into a pry debug console. We can view the
page in the browser and even interact with it and we can also call code in the
console like so.

```
version.text
=> "Rails version: 6.0.0.beta1\nRuby version: ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

from the browser we can inspect that we have

```{html}
<p class="version">
  <strong>Rails version:</strong> 6.0.0.beta1<br>
  <strong>Ruby version:</strong> ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]
</p>
```

we can assert as follows to make sure we have rails version 6 and ruby version 2.6

```
version.text[/(Rails version: )(?<version>[^\n]*)/, "version"]
=> "6.0.0.beta1"
version.text[/(Ruby version: )(?<version>[^\n]*)/, "version"]
=> "ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

which we can implement as

> spec/features/flows/game-app_spec.rb

```{rb}
require 'rails_helper'

feature 'Game App', js: true do
  scenario 'I have a game app' do
    When 'user visits the app' do
      visit('/')
    end

    Then 'user sees they are on rails' do
      wait_for { page }.to have_content("Yay! You’re on Rails!")
      version = page.find("p.version")
      wait_for { version.text[/(Rails version: )(?<version>[^\n]*)/, "version"] }.to match(/^6.0.0/)
      wait_for { version.text[/(Ruby version: )(?<version>[^\n]*)/, "version"]  }.to match(/^ruby 2.6.0/)
    end
  end
end
```

In this case we have written the test after the fact. The aim here is still to get across some core "iteration 0" concepts so bear with us. Ofcourse to confirm the test is actually working it is worth at least changing the expectation to see how it would fail and if that is instructive in how to fix it. Say if we changed it to running ruby 3.0.0

```sh
Failures:

  1) Game App I have a game app
     Failure/Error:
       ...

       Diff:
       -/^ruby 3.0.0/
       +"ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

This seems reasonably readable in that the expected and actual ruby versions are show next to each other even if it does not say that we are expecting to be running a particular version of ruby. We have a passing test and more assertion so we can commit.

COMMIT [confirm rails and ruby version](https://github.com/failure-driven/game-app/commit/5810b82485a5c61f2e694ac6ec06490ced09aaa8)

There is room for refactoring. Although this now passes it is a bit of a mess for a high level flow spec so let's hide the complexity away in a page fragment.

page fragment setup

```
mkdir -p spec/support/features/page_fragments
cat > spec/support/features/page_fragments.rb
module PageFragments
  Page = Struct.new :rspec_example do
    alias_method :browser, :rspec_example # Capybara DSL + rspec example context
  end

  def classify(string)
    string.to_s.split('_').map(&:capitalize).join
  end

  def focus_on(*args)
    require File.join(
      __dir__,
      'page_fragments',
      args.map(&:to_s)
    )
    mod = args.inject(PageFragments) do |klass, sub_klass|
      klass.const_get(classify(sub_klass))
    end
    page = Page.new(self).extend(mod)
    yield page if block_given?
    page
  end
end

# add following to end of spec/rails_helper.rb
  ...
  # include PageFragments in features
  config.include PageFragments, type: :feature
end
```

The basics of page fragments are now setup so we can commit

COMMIT [:wrench: page fragment loading setup](https://github.com/failure-driven/game-app/commit/6e721dd94be18dd0bb2f130b9906e4be0499a67a)

Now we can write a simple page fragment to return the welcome page message and ruby and rails versions.

```
cat > spec/support/features/page_fragments/welcome.rb
module PageFragments
  module Welcome
    def message_and_versions
      output = {}
      output[:message] = browser.find('h1').text
      version = browser.find("p.version")
      output[:rails_version] = version.text[/(Rails version: )(?<version>[^\n]*)/, "version"]
      output[:ruby_version]  = version.text[/(Ruby version: )(?<version>[^\n]*)/, "version"]
      output
    end
  end
end
```

and to use the page fragment

> spec/features/flows/game-app_spec.rb

```{rb}
require 'rails_helper'

feature 'Game App', js: true do
  scenario 'I have a game app' do
    When 'user visits the app' do
      visit('/')
    end

    Then 'user sees they are on rails' do
      wait_for { focus_on(:welcome).message_and_versions }.to include(
        message:       "Yay! You’re on Rails!"
        rails_version: match(/^6.0.0/)
        ruby_version:  match(/^ruby 2.6.0/)
      )
    end
  end
end
```

The failure is not too perfect but good enough. In this case it is more about getting page fragments setup then anything else

```
Failures:
  1) Game App I have a game app
     Failure/Error:
         focus_on(:welcome).message_and_versions
       ...
       Diff:
        :message => "Yay! You’re on Rails!",
       -:rails_version => (match /^6.0.0/),
       -:ruby_version => (match /^ruby 2.7.0/),
       +:rails_version => "6.0.0.beta1",
       +:ruby_version => "ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]",
```

COMMIT [:tada: simplified flow by putting code in page fragment](https://github.com/failure-driven/game-app/commit/e1231c40a768d6f12590030d0cab5f4852a9b776)

# React and Jest

# Rubocop and ESlint

# CI

# CD

# TODO

- GraphQL
- typescript
