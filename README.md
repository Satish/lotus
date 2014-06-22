# Lotus

A complete web framework for Ruby

## Status

[![Gem Version](https://badge.fury.io/rb/lotusrb.png)](http://badge.fury.io/rb/lotusrb)
[![Build Status](https://secure.travis-ci.org/lotus/lotus.png?branch=master)](http://travis-ci.org/lotus/lotus?branch=master)
[![Coverage](https://coveralls.io/repos/lotus/lotus/badge.png?branch=master)](https://coveralls.io/r/lotus/lotus)
[![Code Climate](https://codeclimate.com/github/lotus/lotus.png)](https://codeclimate.com/github/lotus/lotus)
[![Dependencies](https://gemnasium.com/lotus/lotus.png)](https://gemnasium.com/lotus/lotus)
[![Inline docs](http://inch-ci.org/github/lotus/lotus.png)](http://inch-ci.org/github/lotus/lotus)

## Contact

* Home page: http://lotusrb.org
* Mailing List: http://lotusrb.org/mailing-list
* API Doc: http://rdoc.info/gems/lotusrb
* Bugs/Issues: https://github.com/lotus/lotus/issues
* Support: http://stackoverflow.com/questions/tagged/lotus-ruby
* Chat: https://gitter.im/lotus/chat

## Rubies

__Lotus__ supports Ruby (MRI) 2+

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'lotusrb'
```

And then execute:

```shell
$ bundle
```

Or install it yourself as:

```shell
$ gem install lotusrb
```

## Usage

Lotus combines the power and the flexibility of all its [frameworks](https://github.com/lotus).
It uses [Lotus::Router](https://github.com/lotus/router) and [Lotus::Controller](https://github.com/lotus/controller) for routing and controller layer, respectively.
While [Lotus::View](https://github.com/lotus/view) it's used for the presentational logic.

**If you're not familiar with those libraries, please read their READMEs first.**

### Architecture

Unlike the other Ruby web frameworks, it has a flexible conventions for the code structure.
Developers can arrange the layout of their projects as they prefer.
There is a suggested architecture that can be easily changed with a few settings.

Based on the experience on dozens of projects, Lotus encourages the use of Ruby namespaces.
In this way, growing code bases can be split without effort, avoiding monolithic applications.

Lotus has a smart **mechanism of duplication of its frameworks**, that allows multiple copy of a framework and multiple applications to run in the **same Ruby process**.
In other words, even small Lotus applications are ready to be split in separated deliverables, but they can safely coexist in the same heap space.

For instance, when a `Bookshelf::Application` is loaded, `Lotus::View` and `Lotus::Controller` are duplicated as `Bookshelf::View` and `Bookshelf::Controller`, in order to make their configurations completely indepentend from `Backend::Application` thay may live in the same Ruby process.
So that, developers SHOULD include `Bookshelf::Controller` instead of `Lotus::Controller`.

#### One file application

```ruby
# config.ru
require 'lotus'

module OneFile
  class Application < Lotus::Application
    configure do
      routes do
        get '/', to: 'home#index'
      end
    end
  end

  module Controllers::Home
    include OneFile::Controller

    action 'Index' do
      def call(params)
      end
    end
  end

  module Views::Home
    class Index
      include OneFile::View

      def render
        'Hello'
      end
    end
  end
end

run OneFile::Application.new
```

When the application is instantiate, it will also create `OneFile::Controllers` and `OneFile::Views` namespace, to incentivize the modularization of the resources.
Also, note how similar are the names of the action and of the view: `OneFile::Controllers::Home::Index` and `OneFile::Views::Home::Index`.
**This naming system is a Lotus convention and MUST be followed, or otherwise configured**.

#### Microservices architecture

```
test/fixtures/microservices
├── apps
│   ├── backend
│   │   ├── application.rb                  Backend::Application
│   │   ├── controllers
│   │   │   └── sessions.rb                 Backend::Controllers::Sessions::New, Create & Destroy
│   │   ├── public
│   │   │   ├── favicon.ico
│   │   │   ├── fonts
│   │   │   │   └── cabin-medium.woff
│   │   │   ├── images
│   │   │   │   └── application.jpg
│   │   │   ├── javascripts
│   │   │   │   └── application.js
│   │   │   └── stylesheets
│   │   │       └── application.css
│   │   ├── templates
│   │   │   ├── backend.html.erb
│   │   │   └── sessions
│   │   │       └── new.html.erb
│   │   └── views
│   │       ├── backend_layout.rb           Backend::Views::BackendLayout
│   │       └── sessions
│   │           ├── create.rb               Backend::Views::Sessions::Create
│   │           ├── destroy.rb              Backend::Views::Sessions::Destroy
│   │           └── new.rb                  Backend::Views::Sessions::New
│   └── frontend
│       ├── application.rb                  Frontend::Application
│       ├── assets
│       │   ├── favicon.ico
│       │   ├── fonts
│       │   │   └── cabin-medium.woff
│       │   ├── images
│       │   │   └── application.jpg
│       │   ├── javascripts
│       │   │   └── application.js
│       │   └── stylesheets
│       │       └── application.css
│       ├── controllers
│       │   └── sessions
│       │       ├── create.rb               Frontend::Controllers::Sessions::Create
│       │       ├── destroy.rb              Frontend::Controllers::Sessions::Destroy
│       │       └── new.rb                  Frontend::Controllers::Sessions::New
│       ├── templates
│       │   ├── frontend.html.erb
│       │   └── sessions
│       │       └── new.html.erb
│       └── views
│           ├── application_layout.rb       Frontend::Views::ApplicationLayout
│           └── sessions
│               ├── create.rb               Frontend::Views::Sessions::Create
│               ├── destroy.rb              Frontend::Views::Sessions::Destroy
│               └── new.rb                  Frontend::Views::Sessions::New
└── config.ru
```

As you can see, the code can be organized as you prefer. For instance, all the sessions actions for the backend are grouped in the same file,
while they're split in the case of the frontend app.

**This because Lotus doesn't have namespace-to-filename conventions, and doesn't have autoload paths.**
During the boot time it **recursively preloads all the classes from the specified directories.**

```ruby
# apps/backend/application.rb

module Backend
  class Application < Lotus::Application
    configure do
      load_paths << [
        'controllers',
        'views'
      ]

      layout :backend

      routes do
        resource :sessions, only: [:new, :create, :destroy]
      end
    end
  end
end

# All code under apps/backend/{controllers,views} will be loaded
```

```ruby
# config.ru
require_relative 'apps/frontend/application'
require_relative 'apps/backend/application'

run Lotus::Router.new {
  mount Backend::Application,  at: '/backend'
  mount Frontend::Application, at: '/'
}

# We use an instance of Lotus::Router to mount two Lotus applications
```

## Contributing

1. Fork it ( https://github.com/lotus/lotus/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Versioning

__Lotus::Controller__ uses [Semantic Versioning 2.0.0](http://semver.org)

## Copyright

Copyright 2014 Luca Guidi – Released under MIT License
