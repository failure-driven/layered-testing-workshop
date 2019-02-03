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
  ```sh
  brew install rbenv
  rbenv install 2.6.0
  ruby -v
  ```
- node 11.8.0 - https://nodejs.org/en/ 24 Jan 2019
  ```sh
  brew install nvm
  nvm install 11.8.0
  node -v
    v11.8.0
  ```
- rails 6.0.0.beta1 https://rubygems.org/gems/rails/versions 18 Jan 2019
  ```sh
  gem install bundler
  gem install rails --pre
  rails -v
    Rails 6.0.0.beta1
  ```
- postgres
  TODO confirm? create default DB named username? some basic commands?
  ```sh
  brew install postgres
  brew services start postgres
  psql
  ```

## Create new Rails project

Create new Rails 6.0.0.beta1 project, using postgres as data base and RSpec as
testing framework.

TODO - how to make npm default over yarn?

```sh
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

COMMIT [:tada: rails new](https://github.com/failure-driven/game-app/commit/110a9d9229048b62da2371d62f39db2b7bd66d37)

## Add RSpec with capybara, selenium webdriver, and screen shots.

```rb
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
```

```sh
bundle install
bundle          # by default does the install

rails generate rspec:install  # to initialize rspec

rspec
  No examples found.

  Finished in 0.00031 seconds (files took 0.12398 seconds to load)
  0 examples, 0 failures
```

```sh
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
```

```sh
rspec
# Failure
  Failure/Error: raise ActionController::RoutingError, "No route matches [#{env['REQUEST_METHOD']}] #{env['PATH_INFO'].inspect}"

  ActionController::RoutingError:
    No route matches [GET] "/"
```

```sh
# add the default rails/welcome#index
vi config/routes.rb

Rails.application.routes.draw do
  ...
  get '/' => "rails/welcome#index"
end
```

```sh
rspec

# Failure
Failure/Error: wait_for { page }.to have_content("Yay I'm on Rails")
  expected to find text "Yay I'm on Rails" in "Yay! You’re on Rails!\nRails version: 6.0.0.beta1\nRuby version: ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

Assuming we have no idea but we want to move on, we can pend the spec. Just say
that it is in progress, in fact it should actually fail.

> spec/features/flows/game-app_spec.rb

```rb
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

TODO currently COMMIT [:construction: specs setup and a pending spec](https://github.com/failure-driven/game-app/commit/ebde6532f3d4955d6c08de4351c72cccc65c4f25)

- split out the setup from the pending spec

## Configure the browser and screen shots

```sh
vi spec/rails_helper.rb

  7 require 'rspec/rails'
  8 # Add additional requires below this line. Rails is not loaded until this point!
  9
  10 Dir['spec/support/**/*.rb'].each do |file|
  11 require Rails.root.join(file).to_s
  12 end
```

```sh
mkdir -p spec/support
mkdir -p tmp/capybara
```

```rb
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

```rb
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

```sh
    When user visits the app
    Then user sees they are on rails
  I have a game app

Finished in 1.6 seconds (files took 2.89 seconds to load)
1 example, 0 failures
```

COMMIT [passing spec for being on rails](https://github.com/failure-driven/game-app/commit/2f5c9be4f02be0c5b49f44397cb3845cdf3a7039)

TODO again COMMIT [passing speec for being on rails](https://github.com/failure-driven/game-app/commit/2f5c9be4f02be0c5b49f44397cb3845cdf3a7039)

- split out capybara setup from the actual commit

```sh
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

```sh
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

```rb
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

```sh
version.text
=> "Rails version: 6.0.0.beta1\nRuby version: ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

from the browser we can inspect that we have

```html
<p class="version">
  <strong>Rails version:</strong> 6.0.0.beta1<br />
  <strong>Ruby version:</strong> ruby 2.6.0p0 (2018-12-25 revision 66547)
  [x86_64-darwin16]
</p>
```

we can assert as follows to make sure we have rails version 6 and ruby version 2.6

```rb
version.text[/(Rails version: )(?<version>[^\n]*)/, "version"]
=> "6.0.0.beta1"
version.text[/(Ruby version: )(?<version>[^\n]*)/, "version"]
=> "ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]"
```

which we can implement as

> spec/features/flows/game-app_spec.rb

```rb
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

```sh
mkdir -p spec/support/features/page_fragments
```

```rb
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
```

```rb
# add following to end of spec/rails_helper.rb
  ...
  # include PageFragments in features
  config.include PageFragments, type: :feature
end
```

The basics of page fragments are now setup so we can commit

COMMIT [:wrench: page fragment loading setup](https://github.com/failure-driven/game-app/commit/6e721dd94be18dd0bb2f130b9906e4be0499a67a)

Now we can write a simple page fragment to return the welcome page message and ruby and rails versions.

```rb
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

```rb
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

```diff
Failures:
  1) Game App I have a game app
     Failure/Error:
         focus_on(:welcome).message_and_versions
       ...
       Diff:
        :message => "Yay! You’re on Rails!",
-      -:rails_version => (match /^6.0.0/),
-      -:ruby_version => (match /^ruby 2.7.0/),
+      +:rails_version => "6.0.0.beta1",
+      +:ruby_version => "ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin16]",
```

COMMIT [:tada: simplified flow by putting code in page fragment](https://github.com/failure-driven/game-app/commit/e1231c40a768d6f12590030d0cab5f4852a9b776)

# React and Jest

Now that we have Rails setup with integration testing, it is time to add our frontend framework ReactJS. Again even though this is iteration 0 we will still drive it out with some tests. You may have noticed our first test we called `spec/features/flows/game-app_spec.rb`. The idea of a flow is that it flows throught the system and has flow on effects. This is not really what our game spec is doing, it is just checkint that we have testing setup up by checking the default page. To drive out our React tests, rather then creating a flow we will jump down a layer and create a more suitable test known as a page mechanic.

```sh
mkdir -p spec/features/mechanics
```

```rb
cat > spec/features/mechanics/game_spec.rb
require 'rails_helper'

feature 'Game', js: true do
  scenario 'game is a React single page app (SPA)' do
    When 'user visits the game' do
      visit('/game')
    end

    Then 'user sees they are on react' do
      wait_for { focus_on(:game).message_and_versions }.to eq(
        message: 'You are on react',
        react:   '16.7',
        node:    '10.10.0'
      )
    end
  end
end
```

Here we already have the expctation of using a page fragment so let's create one.

```rb
cat > spec/support/features/page_fragments/game.rb
module PageFragments
  module Game
    def message_and_versions
      {
        message: browser.find('span[@data-id="message"]').text,
        react_version: browser.find('span[@data-id="react-version"]').text,
        node_version: browser.find('span[@data-id="node-version"]').text
      }
    end
  end
end
```

as we will be building this up oursleves we can use a simplified way of finding elements. In this case we are using TODO `data-id` (TODO or whatever the recommended way of doing this is) we can also later remove this from our production build artifact if we want to (TODO how do you do that?)

running this brings back an error

```sh
Failures:

  1) Game game is a React single page app (SPA)
     ...
     ActionController::RoutingError:
       No route matches [GET] "/game"
```

but as the general idea of the test is in place we can pend it as per before and commit

```rb
feature 'Game', js: true do
  scenario 'game is a React single page app (SPA)' do
    When 'user visits the game' do
      pending("we don't have route")
      visit('/game')
```

COMMIT [:construction: need to render a React component](https://github.com/failure-driven/game-app/commit/634c10691679edc2d25b8db593ea4542bd02465e)

To utilise some rails generator magic we will create a GameController with an index action, a view and a basic test.

```sh
bundle exec rails generate controller GameController index --no-assets --skip-routes --no-helper --no-view-specs

      create  app/controllers/game_controller.rb
      invoke  erb
      create    app/views/game
      create    app/views/game/index.html.erb
      invoke  rspec
      create    spec/controllers/game_controller_spec.rb
```

Now we can add a specific route to `config/routes.rb` file like

```rb
Rails.application.routes.draw do
  ...
  get '/game' => 'game#index'
end
```

Running the tests still fails as we do not have the required ReactJS information but no longer for the missing route. We can move our pending message down in `spec/features/mechanics/game_spec.rb`

```diff
    When 'user visits the game' do
-     pending("we don't have route")
      visit('/game')
    end

    Then 'user sees they are on react' do
+     pending("We're not rendering a ReactJS component")
      ...
```

And running all the specs shows us the controller spec and the failing mechanic

```sh
rspec
  GameController
  GET #index
  returns http success

Game App

  When user visits the app
  Then user sees they are on rails
  I have a game app

Game
  When user visits the game
  Then user sees they are on react (FAILED)
  game is a React single page app (SPA) (PENDING: we don't have route)
```

We are good to commit

COMMIT [:construction: add controller to hold our react SPA](https://github.com/failure-driven/game-app/commit/1a0f8ab268f3dc6ffc2f014bcfd26413c4f355aa)

Rails 6 comes with `webpacker` gem by default. This means we can use it to install react for us.

```sh
# looking at all the things that webpacker can do
bundle exec rails --tasks webpacker
  rails webpacker                     # Lists all available tasks in Webpacker
  rails webpacker:binstubs            # Installs Webpacker binstubs in this application
  rails webpacker:check_binstubs      # Verifies that webpack & webpack-dev-server are present
  rails webpacker:check_node          # Verifies if Node.js is installed
  rails webpacker:check_yarn          # Verifies if Yarn is installed
  rails webpacker:clobber             # Remove the webpack compiled output directory
  rails webpacker:compile             # Compile JavaScript packs using webpack for production with digests
  rails webpacker:info                # Provide information on Webpacker's environment
  rails webpacker:install             # Install Webpacker in this application
  rails webpacker:install:angular     # Install everything needed for Angular
  rails webpacker:install:coffee      # Install everything needed for Coffee
  rails webpacker:install:elm         # Install everything needed for Elm
  rails webpacker:install:erb         # Install everything needed for Erb
  rails webpacker:install:react       # Install everything needed for React
  rails webpacker:install:stimulus    # Install everything needed for Stimulus
  rails webpacker:install:typescript  # Install everything needed for Typescript
  rails webpacker:install:vue         # Install everything needed for Vue
  rails webpacker:verify_install      # Verifies if Webpacker is installed
  rails webpacker:yarn_install        # Support for older Rails versions
```

```sh
rails webpacker:install:react
```

this modifies some files

```sh
modified:   babel.config.js
modified:   config/webpacker.yml
modified:   package.json
modified:   yarn.lock
```

and creates a demo component `app/javascript/packs/hello_react.jsx` this can be rendered in our game view `app/views/game/index.html.erb` with the following

```rb
<%= javascript_pack_tag 'hello_react' %>
```

Let's commit before we change this to what we want.

COMMIT [:tada: react render in rails view](https://github.com/failure-driven/game-app/commit/64a64ec40084f60f84149007534f2bf51bf37d28)

## Adding Jest

Now we want to drive out behaviour at the React component test to show the version of react which is being used. We do this with Jest test framework. To add this to the project we do the following.

```sh
yarn add jest babel-jest --save-dev
```

and create a basic jest unit test

```js
cat > app / javascript / packs / game.test.js;
import React from 'react';

describe('game', () => {
  it('we have jest tests', () => {
    expect(true).toEqual(true);
  });
});
```

configure `package.json` to be able to run jest tests

```json
  ...
  "scripts": {
    "test": "jest"
  }
```

which can now be run with

```sh
jest test
```

or if we want to keep the tests running we pass the `--watchAll` flag to the jest program using the `--` flag

```sh
jest test -- --watchAll
```

Here we see there is a problem as there is a `config/webpack/test.js` file that matches the default test file matcher for Jest `/test\.js/` to ignore this we can do that through a `jest.config.js` file as follows

```js
cat > jest.config.js;
module.exports = {
  testPathIgnorePatterns: ['config/webpack']
};
```

now we have Jest setup with 1 passing test and we are ready to commit

COMMIT [:wrench: jest test setup](https://github.com/failure-driven/game-app/commit/036a3ad22e2f0dc3934803faf4d4b68a11e26bd8)

## Enzyme for React Component testing

Although Jest has a way of testing react comonents https://jestjs.io/docs/en/tutorial-react the de-facto standard is to use Airbnb's enzyme. Let's set that up.

```sh
yarn add enzyme enzyme-adapter-react-16 --save-dev
```

we can now update our demo Jest test `app/javascript/packs/game.test.js` to actually test a React component.

```js
import React from 'react';
import { configure, shallow } from 'enzyme';
import Game from './game';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });

describe('Game', () => {
  it('renders a <div>', () => {
    const wrapper = shallow(<Game />);
    expect(wrapper.type()).toBe('div');
    expect(wrapper.prop('children')).toBe('16.7.0');
  });
});
```

This will require that our game component returns a `div` with the text child component to be the react version number, `16.7.0`. Running this will fail until you implement this with something like the following.

```js
import React from 'react';
const Game = props => <div>{React.version}</div>;
export default Game;
```

COMMIT [:tada: a first component tested with enzyme](https://github.com/failure-driven/game-app/commit/f6ef499c6ba3bd41c78c19a318b2e8473f8d91c5)

Probably best to move the enzyme config to a central `app/javascript/jestSetup.js` file

TODO confirm above location for jestSetup file

```js
cat > jestSetup.js;
import Adapter from 'enzyme-adapter-react-16';
import { configure } from 'enzyme';

configure({ adapter: new Adapter() });
```

and this generic code can be removed from `app/javascript/packs/game.test.js`

COMMIT [:wrench: add enzyme setup to jestSetup.js](https://github.com/failure-driven/game-app/commit/407928faa2a26057f8aa59cc89492fe354e3624f)

Now removing the demo `hello_react.jsx` and wiring up game to be rendered to `game#index`

COMMIT [wire up game to be rendered by game#index'](https://github.com/failure-driven/game-app/commit/bb94c10e2ac0dc7d3956052e6930cd8f2bd38c24)

Test driving out the component to give us the react message and version gives us this for the test

```js
describe('Game', () => {
  let wrapper;

  beforeEach(() => {
    wrapper = shallow(<Game />);
  });

  it('renders a <div>', () => {
    expect(wrapper.type()).toBe('div');
  });

  it('renders a div with message and one with version as children', () => {
    expect(wrapper.find('[data-test="message"]').prop('children')).toBe(
      'You are on React'
    );
    expect(wrapper.find('[data-test="react-version"]').prop('children')).toBe(
      '16.7.0'
    );
  });
});
```

and this for the implementation

```js
const Game = props => (
  <div>
    <div data-test="message">You are on React</div>
    <div>
      React Version: <span data-test="react-version">{React.version}</span>
    </div>
  </div>
);
```

Now our react component test passes as does the assertion in our game mechanic so we can un pend that and we are all green

COMMIT [:tada: we have react SPA](https://github.com/failure-driven/game-app/commit/8caa60b2af0292b0e2604c84e11fc32fed19b19b)

As the react expectations are not doing anything clever around passing variables down to other components or dealing with state, we can replace the tests with a enzyme snapshot test.

First we will install and setup `enzyme-to-json`

```sh
yarn add enzyme-to-json --save-dev
```

adding to `jest.config.js` the following line

```js
  snapshotSerializers: ['enzyme-to-json/serializer'],
```

now all the other tests in game.test.js can be replaced with the snapshot

```js
it('renders correctly', () => {
  expect(wrapper).toMatchSnapshot();
});
```

and the snapshot looks like this

```js
cat app/javascript/packs/__snapshots__/game.test.js.snap
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`Game renders correctly 1`] = `ReactWrapper {}`;
```

COMMIT [:confused: added snapshot testing but nothing is in the snapshot](https://github.com/failure-driven/game-app/commit/c71ac622a3372772754b50732ecbeb2bcc5f80fd)

TODO maybe the snapshot is not good in this case

TODO auto reload for webpack https://github.com/rails/webpacker#development

TODO how to remove `data-test` attributes

TODO rails generator to get to here

## Rubocop, ESlint and prettier

Before we get too caried away writing any code we should make sure we can maintain our code quality with tools like Rubcop and ESLint.

Adding `rubocop` as a gem dependency

```rb
group :development, :test do
  ...
  gem 'rubocop'
end
```

we can now run `rubocop` but there will be a number of errors we don't care to much about like files that have been mostly generated like those in `config/` directory, using the `.rubocop.yml` config file as per next commit

COMMIT [:construction: basic rubocop setup]()

Now running rubocop and applying the fixes automatically we have a more consistent code base.

```sh
rubocop --auto-correct
rubocop -a             # shorter version
```

COMMIT [:wrench: adhere to rubocop recommendations]()

TODO how to auto save with rubocop recommendations?
TODO git pre commit hook for lines changed only

For ESlint

```sh
yarn add eslint eslint-plugin-react babel-eslint eslint-plugin-jest --save-dev
```

```js
// package.json
"scripts": {
  "test": "jest",
  "lint": "eslint"
}
```

```js
// .eslintrc.js
module.exports = {
  plugins: ['react', 'jest'],
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:jest/recommended'
  ],
  parser: 'babel-eslint',
  env: {
    browser: true
  },
  rules: {
    quotes: ['error', 'single', { avoidEscape: true }],
    'comma-dangle': ['error', 'always-multiline']
  }
};
```

COMMIT [:wrench: configure eslint]()

```sh
yarn lint -- --fix app/javascript/
```

and some js fixes

COMMIT [:wrench: adhere to eslint]()

also running tests at this time shows that snapshot tests work?

```diff
// Jest Snapshot v1, https://goo.gl/fbAQLP

-exports[`Game renders correctly 1`] = `ShallowWrapper {}`;
+exports[`Game renders correctly 1`] = `
+<div>
+  <div
+    data-test="message"
+  >
+    You are on React
+  </div>
+  <div>
+    React Version:
+    <span
+      data-test="react-version"
+    >
+      16.7.0
+    </span>
+  </div>
+</div>
+`;
```

COMMIT [:confused: snapshot tests now work?]()

## CI

## CD

## TODO

- GraphQL
- typescript

```

```
