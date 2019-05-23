---
layout: post
title: Starting a Selenium Server inside TravisCI
comments: true
spotifyTrack: spotify:track:6CmsmQkg655kIZmg22IlVD

---

**TL;DR:** To start a Selenium Server, namely a Chrome Driver, inside TravisCI builds you have to install and run a Google Chrome instance using your .travis.yml configuration, like in this working example:
{:.message}


```yml
sudo: required
dist: trusty
cache:
  directories:
    - node_modules
language: node_js
node_js:
          - '8'
          - '7'
          - '6'
# setting up environment variables
env:
  global:
    - CXX=g++-4.8
    - DISPLAY=:99.0
    - CHROME_BIN=/usr/bin/google-chrome
# installing Chrome for e2e tests
addons:
  apt:
    sources:
      - google-chrome
    packages:
      - google-chrome-stable
      - oracle-java9-set-default
# the following step should start the virtual X frame buffer: Xvfb process
before_script:
  - "sh -e /etc/init.d/xvfb start"
  - sleep 3 #wait a while before xvfb to start
after_script:
  - "sh -e /etc/init.d/xvfb stop"
```

The above listing runs three CI builds (using node.js versions 6, 7 and 8) for a Javascript application containing [Nightwatch.js](http://nightwatchjs.org) end to end tests.

## My use case scenario

I am working on a [Vue client for Freedomotic](https://github.com/freedomotic/fd-vue-webapp), a development framework for IoT. In this Vue project there was an issue requiring to configure a Travis CI script for the project, since the original one did not work properly.

Indeed, the first `.travis.yml` script was the following:

```yml
language: node_js
node_js:
          - '8.8.1'
```

As you may notice, it is a very basic build file. Alas, it makes the build failing during the end to end test phase. The error message is the following:

```
Running:  default e2e tests
Error retrieving a new session from the selenium server
Connection refused! Is selenium server started?
{ Error: socket hang up
    at createHangUpError (_http_client.js:329:15)
    at Socket.socketCloseListener (_http_client.js:361:23)
    at emitOne (events.js:120:20)
    at Socket.emit (events.js:210:7)
    at TCP._handle.close [as _onclose] (net.js:557:12) code: 'ECONNRESET' }
```   

By investigating a bit, I understood that the above failure happens because of a not starting Selenium Server, that is required by the scaffolded application to run the end to end tests (built with Nightwatch.js).  
By default, the e2e tests configuration that comes with the scaffolding of Vue applications (see [vue-cli](https://github.com/vuejs/vue-cli)), uses [Chrome Driver](https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver) as WebDriver for Selenium, so to start the Selenium server properly, the Travis CI build script needs to setup a Google Chrome instance before tests execution.

## Updating .travis.yml

In order to run a ChromeDriver Selenium Server in TravisCI it is required to:

 **1** - flag the build machine to run with `sudo` option enabled:

```yml
  sudo: required
```

 **2** - use a valid Ubuntu distribution (in my case `trusty`):

```yml
  dist: trusty
```

**3** - define a set of environment variables. In detail they are:

  - `CXX`, that sets the program for compiling C++ programs to the 4.8 version of ‘g++’;
  - `DISPLAY`, that it is the port where the Virtual X frame Buffer display is opened;
  - `CHROME_BIN`, that is the location of the binary running Chrome  

 **4** - install the required Google Chrome packages:

```yml
addons:
  apt:
    sources:
      - google-chrome
    packages:
      - google-chrome-stable
      - oracle-java8-set-default
```

**5** - run the [Virtual X frame Buffer](https://en.wikipedia.org/wiki/Xvfb) before the execution of the tests (and make it sleeping for a couple of seconds), like this:

```yml
this:before_script:
  - "sh -e /etc/init.d/xvfb start"
  - sleep 3
```

**6** - for completeness stop the Virtual X frame Buffer after the tests execution.

So, the final version of my `.travis.yml` file looks like the one at the top of this post.

## Final considerations

Although the above file has been just tested for a node.js application, my assumption here is that all the steps described in the previous section are useful also to setup a valid running Selenium Server for projects implemented using other programming languages (e.g.: Java).  
However, eventually I came across a [documentation section](https://docs.travis-ci.com/user/gui-and-headless-browsers/) of Travis that helped me to make this script leaner than the one above. Long story short, the following is a very clear  `.travis.yml` to allow a Google Chrome instance running before the required e2e tests:

```yml
dist: trusty
cache:
  directories:
    - node_modules
language: node_js
addons:
  chrome: stable
node_js:
          - '8'
          - '7'
          - '6'
before_script:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start" # the starting the virtual X frame buffer: Xvfb process
  - sleep 3 #wait a while before xvfb to start
after_script:
  - "sh -e /etc/init.d/xvfb stop"
```
