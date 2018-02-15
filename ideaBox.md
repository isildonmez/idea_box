- The `Gemfile` is just a text file within your project which contains a list of gems for use in your application.

# IdeaBox Project from [jumpstartlab](http://tutorials.jumpstartlab.com/projects/idea_box.html#ideabox)

## I0: Getting Started

It is used to express the dependencies of our application. We need the Sinatra library for our app to work, so we add it to the `Gemfile`:

```
source 'https://rubygems.org'

gem 'sinatra', require: 'sinatra/base'
```

In the `app.rb` we’ll start writing the body of our application.

```
require 'bundler'
Bundler.require

class IdeaBoxApp < Sinatra::Base
  get '/' do
    "Hello, World!"
  end

  run! if app_file == $0
end
```

### What just happened?

```
require 'bundler'
Bundler.require
```
Here, we loaded Bundler and told it to require everything specified in the `Gemfile`.

```
class IdeaBoxApp < Sinatra::Base
end
```
Then we create a Ruby class which is going to be our application. It inherits from `Sinatra::Base`, which provides the application with all the Sinatra functionality.

```
class IdeaBoxApp < Sinatra::Base
  get '/' do
  end
end
```

We call a method named `get`, giving it a parameter `/` and a block of code. The string parameter to `get` is the URL pattern to match against the incoming HTTP request.

Our Sinatra application knows that it is running on http://localhost:4567, so we only need to define the part of the url that comes after that.

In this case we passed '/', which will match when someone visits the homepage, also known as the *root* of the application.

```
class IdeaBoxApp < Sinatra::Base
  get '/' do
    'Hello, world!'
  end
end
```

The block of code that we pass to the `get` method, the stuff between the `do` and the `end`, is the code that Sinatra is going to run when someone requests the matching page.

The response to the request is a string, in this case 'Hello, World!', which is sent to the browser.

```
class IdeaBoxApp < Sinatra::Base
  # ...

  run! if app_file == $0
end
```
This functionality will be important later. Basically we only want to call `run!`, which actually starts the application, if this file was run directly like this:

```
ruby app.rb
```
It’s not a very good idea to have the app run itself. That line `run! if app_file == $0` is ugly. Let’s separate the concerns of *defining* the application from *running* the application.

#### Rack, in brief

There’s a standard named Rack that’s used by most Ruby web frameworks, including both Sinatra and Rails. It’s a small interface that each framework follows.

This allows the community to share tools across frameworks. The Puma web server, for instance, supports Rack applications. So that means it can run either Sinatra or Rails apps without knowing anything more than the fact that they adhere to the Rack interface.

#### config.ru

Rack applications have a file in their project root named `config.ru`. When a Rack-compatible server is told to load the application, it’ll try to run this file. Despite having the extension `.ru`, it’s just another Ruby file.

`config.ru` is called a *rack up* file, hence the `ru` file extension. We’re going to move the business of actually running the application into that file.

#### Default Ports

Earlier when we started the application directly it started on port 4567. In our browser we opened `localhost:4567`.

`rackup` defaults to the port `9292` instead of port `4567`, so we’d access the page at `localhost:9292`.

You can pick whatever port you like, and tell `rackup` to use it. Let’s stick with Sinatra’s default, `4567`:

```
rackup -p 4567
```

HTML is really just a string with some special tags in it. The browser understands that tag structure and has opinions about what it should look like.

Putting our HTML in the block of the `get` method works, but it’s difficult to read, write, and maintain. We need a better way to render the response: creating a view template.

#### Reloading Application Code

If you’ve developed anything in Rails before, you’ve been spoiled by automatic reloading. In Ruby it’s actually pretty complex to dynamically undefine and redefine classes when files change. In Sinatra, this functionality is **not** built in by default.

##### Sinatra Reloader

One to get automatic reloading is to use the `sinatra/reloader` functionality of the `sinatra-contrib` gem.

Add `sinatra-contrib` to your `Gemfile`:

```
gem 'sinatra-contrib', require: 'sinatra/reloader'
```
Then run `bundle` from your terminal to install the gem.
Add this `configure` block into your `app.rb`:

```
class IdeaBoxApp < Sinatra::Base
  configure :development do
    register Sinatra::Reloader
  end

  # ... other stuff
end
```

##### Shotgun

Shotgun is an automatic reloader for Rack. It works with any Rack-supported server. According to the documentation, "Each time a request is received, it forks, loads the application in the child process, processes the request, and exits the child process. The result is clean, application-wide reloading of all source files and templates on each request."

To use Shotgun, add it to your Gemfile:

```
gem 'shotgun'
```

Then from the command line, run `shotgun`.









