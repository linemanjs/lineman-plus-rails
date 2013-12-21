# lineman + rails

Behold, a Rails app that can build and deploy lineman assets easily. Under this
approach, an application's JavaScript & CSS would be built by lineman but would
flow through the asset pipeline like any other asset in a Rails application.

This gives you the productivity and organizational benefits of separating out your
front-end source code with a first-class build tool in Lineman, but with the convenience
of fitting into the typical Rails deployment story.

## Setup

### the Rails side

Add 'rails-lineman' to your Gemfile.

``` ruby
gem 'rails-lineman'
```

And configure the app to find your Lineman project. In this repo, that's done in `config/application.rb`:

``` ruby
config.rails_lineman.lineman_project_location = "my-lineman-app"
```

Alternatively, rails-lineman will look for an environment variable named `LINEMAN_PROJECT_LOCATION`.

### the Lineman side

Just add the lineman-rails plugin to your project:

```
$ npm install --save-dev lineman-rails
```

This will set up some additional static routes and enable the API proxy.
It will also disable the pages task.

## Development

When you're developing you'll want to start the Rails app:

```
$ bundle exec rails s
```

And separately start the Lineman app, which will proxy back to the Rails app:

```
$ lineman run
```

You'll actively develop on Lineman's port (default is [localhost:8000](http://localhost:8000))
so that you'll be able to access the JS & CSS you're developing under Lineman as if it were layered
on top of your Rails views & APIs.

Speaking of Rails views, whenever you want to include the Lineman JS & CSS in a Rails view or layout,
just include it like this in your ERB (in this app, see `app/views/widgets/index.html.erb`):

``` erb
<%= stylesheet_link_tag "lineman/app" %>
<%= javascript_include_tag "lineman/app" %>
```

## Production

At deploy time, the `rails-lineman` gem will wrap the `assets:precompile task such
that the Lineman assets are first built (by default, into
`app/assets/javascripts/lineman` and `app/assets/stylesheets/lineman`). Then the
asset pipeline will compile all of your assets as it normally would into
`public/assets`. Finally, the Lineman assets are removed from the app so they
don't clutter up your version control.

## Issues

At this time you can't push this repo to heroku because apparently `npm` isn't
on the path on the cedar VM. Whoops.

```
-----> Preparing app for Rails asset pipeline
       Running: rake assets:precompile
       rake aborted!
       rails-lineman failed while running `npm install` from the `/tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/my-lineman-app` directory.
       Make sure that you have Node.js installed on your system and that `npm` is on your PATH.
       You can download Node.js here: http://nodejs.org
       /tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/vendor/bundle/ruby/2.0.0/gems/rails-lineman-0.0.1/lib/rails_lineman/lineman_doer.rb:57:in `run_npm_install'
       /tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/vendor/bundle/ruby/2.0.0/gems/rails-lineman-0.0.1/lib/rails_lineman/lineman_doer.rb:16:in `block in build'
       /tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/vendor/bundle/ruby/2.0.0/gems/rails-lineman-0.0.1/lib/rails_lineman/lineman_doer.rb:51:in `chdir'
       /tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/vendor/bundle/ruby/2.0.0/gems/rails-lineman-0.0.1/lib/rails_lineman/lineman_doer.rb:15:in `build'
       /tmp/build_9ab8fe0d-b46f-4899-9d47-3c673bc6c156/vendor/bundle/ruby/2.0.0/gems/rails-lineman-0.0.1/lib/tasks/assets_precompile.rake:9:in `block (2 levels) in <top (required)>'
       Tasks: TOP => assets:precompile
       (See full trace by running task with --trace)
```
