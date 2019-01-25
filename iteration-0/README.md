# Iteration 0 - WebApp, CI/CD

> As a developer

> I want a web app setup with CI/CD

> So that I have delivery to the web

The aim of iteration 0 is to have all the code and tooling in
place to be able to regularily and confidently deploy to
production. These means setting up:
* The code frameworks - Rails and ReactJS
* Server configuration for servers, databases, etc
* a basic test suite
* A CI server to run the tests
* A CD script to push code to production
* Production or some kind of hosting.

In our example we will use Rails as the framework for the backend
and ReactJS as the Single Page App (SPA) frontend. We will use
RSpec as the main integration and rails testing framework and
Jest for frontend tests. This will be configured to work with a
free tier [Travis CI | Build Kite | Semaphore CI | other].
Finally we will deploy to a free tier Heroku app.

If there are any issues configuring this there will also be the
ability to use vagrant to use a pre configured machine setup.

Options for using the latest and greatest

