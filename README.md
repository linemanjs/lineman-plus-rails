# lineman + rails

Behold, a Rails app that can build and deploy [Lineman](http://linemanjs.com) assets easily. Under this
approach, an application's JavaScript & CSS would be built by lineman but would
flow through the asset pipeline like any other asset in a Rails application.

This gives you the productivity and organizational benefits of separating out your
front-end source code with a first-class build tool in Lineman, but with the convenience
of fitting into the typical Rails deployment story.

This repo was pushed to Heroku using the default Rails buildpack and can be
demoed [here](http://mysterious-retreat-8708.herokuapp.com/widgets).

## Setup

Instructions on how to reproduce this repo's setup follows:

### the Rails side

Add [rails-lineman](https://github.com/linemanjs/rails-lineman) to your Gemfile.

``` ruby
gem 'rails-lineman'
```

And configure the app to find your Lineman project. In this repo, that's done in `config/application.rb`:

``` ruby
config.rails_lineman.lineman_project_location = "my-lineman-app"
```

Alternatively, rails-lineman will look for an environment variable named `LINEMAN_PROJECT_LOCATION`.

### the Lineman side

Just add the [lineman-rails](https://github.com/linemanjs/lineman-rails) plugin to your project:

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

If you're cloning this repo to play around, try hitting: [localhost:8000/widgets](http://localhost:8000/widgets)
and you should see Lineman's familiar hello world example render properly:

![Screenshot](http://f.cl.ly/items/2z32393h001w1a3S0K3f/Screen%20Shot%202013-12-21%20at%2016.01.23.png)

## Production

At deploy time, the `rails-lineman` gem will wrap the `assets:precompile` task such
that the Lineman assets are first built (by default, into
`app/assets/javascripts/lineman` and `app/assets/stylesheets/lineman`). Then the
asset pipeline will compile all of your assets as it normally would into
`public/assets`. Finally, the Lineman assets are removed from the app so they
don't clutter up your version control.

That is to say, when you precompile assets:

```
$ RAILS_ENV=production bundle exec rake assets:precompile
```

You'll see output include your Lineman bundles:

```
#...
I, [2013-12-21T16:06:11.354975 #44655]  INFO -- : Writing /Users/justin/code/vagrants/oss/projects/javascript/lineman-plus-rails/my-rails-app/public/assets/lineman/app-e3428c9a812e9ed915d331ea97e88357.js
I, [2013-12-21T16:06:11.358738 #44655]  INFO -- : Writing /Users/justin/code/vagrants/oss/projects/javascript/lineman-plus-rails/my-rails-app/public/assets/lineman/app-fb85a2e2e5fd3d180c60a66772648703.css

#...
```

Such that the `public/assets` folder looks like:

```
├── application-76c1a0ca10a36eee991a7b35233c7a93.js
├── application-76c1a0ca10a36eee991a7b35233c7a93.js.gz
├── application-fb85a2e2e5fd3d180c60a66772648703.css
├── application-fb85a2e2e5fd3d180c60a66772648703.css.gz
├── lineman
│   ├── app-e3428c9a812e9ed915d331ea97e88357.js
│   ├── app-e3428c9a812e9ed915d331ea97e88357.js.gz
│   ├── app-fb85a2e2e5fd3d180c60a66772648703.css
│   └── app-fb85a2e2e5fd3d180c60a66772648703.css.gz
└── manifest-c468cd8d56c4364af6110bc39b00ed40.json
```

Note that in Rails 4, Rails will by default not serve any static assets out of
`public/assets` when running in the production environment (leaving that up to
Apache/nginx). To verify this is all working you may need to configure in
`config/environments/production.rb`:

``` ruby
config.serve_static_assets = true
```

## Heroku

Somewhat humorously, rails-lineman will actually try to detect whether it's running on Heroku
(by looking for STACK and DYNO environment variables) and will install Node.js and add it to
the path so that it can build the lineman project.
