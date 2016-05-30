---
layout: post
type: article
title:  "Using Travis CI with Haskell Stack"
author: "Christopher Randall Wells"
image: ""
date:   2015-05-28 21:45:00
categories: github travis-ci haskell stack
---
When I first decided to try switching from using Cabal to Stack for Haskell development, one of the main concerns that I had was with how it would work with Travis CI. I always try to use a CI service with any project I work on in order to double check forany issues that may arise durring development, so I always try to make sure that whatever build tool I use is compatable with the CI service I am using.

Travis CI, by default, uses Cabal when working with Haskell projects. Unfortunately it is not yet able to directly detect and work with Stack projects. A quick search revieled [this page](http://docs.haskellstack.org/en/stable/travis_ci/) on using Stack with Travis CI, however the first time I looked over it I found it difficult to follow. In addition, that page did not cover some aspects of CI services that I use regularly, such as coverage reporting.

After trying a few different things, I was able to come up with a `.travis.yml` file setup that worked for Stack Haskell projects. Here I will show and explain in detail the configuration file I setup and why it is setup the way it is.


## General Description
The Travis CI configuration file needed to be able to do the following things in order to meet my needs:

- Build and test the Haskell project using Stack rather than Cabal
- Generate the project's documentation using Stack to check for errors in documentation markup
- Check to make sure that the package is valid
- Send coverage reports on the unit tests of the package to Coveralls.io

## Installation Steps
In a Travis CI build config, the `before_install` and `install` steps are used to dependencies required for building and testing a project.

Specifically, the `before_install` step is used for installing more general dependencies such as packages. While, the `install` step is used for installing things like libraries that the project depends on.

### before_install
In the `before_install` step, Stack should be installed, along with the required version of GHC and any service programs, such as those that handle coverage reports.

In the config file I put together, the first thing I do in the `before_install` step is download Stack. To do this, I used the process that the Haskell Stack documentation [reccomends using](http://docs.haskellstack.org/en/stable/travis_ci/#installing-stack). Which simply creates a user bin to store the Stack executable, adds that directory to the path, and downloads the Stack executable from Stackage.

```
# Download and unpack the stack executable
- mkdir -p ~/.local/bin
- export PATH=$HOME/.local/bin:$PATH
- travis_retry curl -L https://www.stackage.org/stack/linux-x86_64 | tar xz --wildcards --strip-components=1 -C ~/.local/bin '*/stack'
```

After downloading Stack, you want to download the proper version of GHC, as well as any service programs. In this case I wanted to install `stack-hpc-coveralls` in order to handle sending test coverage reports to Coveralls.io.`

Downloading the proper version of GHC is handled by Stack's `setup` subcommand, so you just need to run that. In this case I run it and any other Stack commands with the `--no-terminal` flag in order to make the output easier to read in Travis CI's build logs. After that, stack-hpc-coveralls can be installed through Stack as well.

```
# Setup Stack and install dependencies for tools
- stack setup --no-terminal
- stack install stack-hpc-coveralls --no-terminal
```

### install
In the `install` step, normally the project dependencies would be installed. However, I am not sure how this can be done with Stack. In Cabal you can do this by running `cabal install --only-dependencies`, but I have been unable to find out the equivalent command in Stack. Instead, the project dependencies can be automatically installed durring the build process.

By defualt, Travis CI [runs the following command durring the install step](https://docs.travis-ci.com/user/languages/haskell#Travis-CI-uses-cabal): `cabal install --only-dependencies --enable-tests`. However, since we are using Stack we do not want it to run that command. Thus we need to override the default `install` step. This can be done by giving a simple echo command in the install step, which doesn't do anything important other than overriding the default step.

```
install:
  # Overwrite the default installation command used by Travis CI
  - echo "Dependencies are installed by Stack in the build process."
```

## Chaching

## The Code
Here is the final resulting Travis CI configuration file I ended up putting together:

```
language: haskell

sudo: false

cache:
  directories:
    - $HOME/.stack

ghc:
  - 7.8

before_install:
  # Download and unpack the stack executable
  - mkdir -p ~/.local/bin
  - export PATH=$HOME/.local/bin:$PATH
  - travis_retry curl -L https://www.stackage.org/stack/linux-x86_64 | tar xz --wildcards --strip-components=1 -C ~/.local/bin '*/stack'

  # Setup Stack and install dependencies for tools
  - stack setup --no-terminal
  - stack install stack-hpc-coveralls --no-terminal

install:
  # Overwrite the default installation command used by Travis CI
  - echo "Dependencies are installed by Stack in the build process."

script:
  # Build and test the program
  - stack build --no-terminal --coverage
  - stack test --no-terminal --coverage

  # Install and run the system tests of the program, using the program
  - stack install --no-terminal
  - system-test tests/system/tests.json

  # Generate the documentation and check the package validity
  - stack haddock --no-terminal
  - cabal check

after_script:
  # Send a test coverage report to Coveralls.io
  - shc system-test unit-tests
```
