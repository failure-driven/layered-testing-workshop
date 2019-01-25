# Outside in, multi layered BDD

A workshop on outside in, multi layered, behavioural driven development (BDD)

# Introduction

  * TODO

# iteration 0 - WebApp, CI/CD

_Story_

> As a developer

> I want a web app setup with CI/CD

> So that I have delivery to the web

* **framework setup**
  * rails new, webpack, rubocop, eslint
  1. basic:
    1. rails new
    1. create-react-app
  1. basic with webpacker?
  1. all out latest:
    1. gem install rails -pre
    1. rails new
    1. webpacker setup

       npm install react@16.7.0-alpha.0 --save
       npm install react-dom@16.7.0-alpha.0 --save

  1. API vs GraphQL ?
    * TODO

* **testing setup**
  * rspec
  * rspec integration selenium
  * rspec flows
  * rspec page fragments
  * jest
  * ci test all script

* **vagrant fallback**
  * TODO

* **CI/CD pipeline**
  * TODO

# Story 1 - first feature - images on a page

> As an internet user

> I want the game to display images

> which will keep me interested and guessing as to what term was used to create them

* Introduction to all the layers
  * “Lifecycle Flow”
    - GIVEN term of "flowers",
    - WHEN I visit the game,
    - THEN I see images of flowers
  * “Domain Unit Tests”
    - term
  * “Lifecycle Flow” - still failing
  * “Page/Component Mechanics”
    - WHEN I visit the game
    - THEN status of loading
    states of image loading, images ready to show, images displayed
  * “Lifecycle Flow” - still failing
  * “Frontend Component Tests”
    - show status of loading
  * “Page/Component Mechanics”
    - WHEN images loaded
    - THEN status of images loaded
  * “Frontend Component Tests”
    - show status of loaded
  * “API Acceptance Tests”
    - fetch images
  * “Controller Unit Tests”
    - call image fetcher return the results
  * “External Integration Tests”
    - make a call to fetch
  * “Domain Concept Tests”
  * “Lifecycle Flow” - now failing failing

**Retro**

- all the layers
  * “Lifecycle Flow”
  * “Page/Component Mechanics”
  * “Frontend Component Tests”
  * “API Acceptance Tests”
  * “Request Tests” - TODO example
  * “Controller Unit Tests”
  * “External Integration Tests”
  * “Domain Concept Tests” - TODO example
  * “Domain Unit Tests”
- how to choose layers
- what to expect to fail where
- why the layer, vs units, vs integrations - what is the unit under test

# Story 2 - guess the images

  * TODO

# Story 3 - multi round game

  * TODO

# Story 4 - admin setup

  * TODO

# Story 5 - users

  * TODO

# Story 6 - score board

  * TODO

# Story 7 - profit?

- expand to cover missing concepts?
- add payment gateway?

