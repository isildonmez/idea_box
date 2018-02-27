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

#### Creating a Form

Sinatra’s default `404` page, which is rendered whenever you submit a request which doesn’t have a matching route.

Typically it means you:

1. Didn’t define the route pattern at all
2. Your method is using the wrong HTTP verb (ex: your method uses get, but the request is coming in as a post)
3. The route pattern doesn’t match the request like you thought it did

##### Create error page

The default error page is pretty useless for debugging. Let’s create a better error page by defining `not_found` method in `app.rb`.

##### Handling POST requests to /

What should our POST / path actually do? Let’s write some pseudocode:

```
post '/' do
  # 1. Create an idea based on the form parameters
  # 2. Store it
  # 3. Send us back to the index page to see all ideas
  "Creating an IDEA!"
end
```

## I2: Saving Ideas

How should we save our data? Should we store it in memory, to a file, or to a database?

Almost every web application is backed by a database that stores its data. We’re going to use an incredibly simple database that comes with Ruby called `YAML::Store` to store our ideas.

```
def save
  database.transaction do |db|
    db['ideas'] ||= []
    db['ideas'] << {title: 'diet', description: 'pizza all the time'}
  end
end
```
Here our call to `database` is returning an instance of `YAML::Store`. The `YAML::Store` instance has a method named `transaction`. A transaction is a set of database operations that are grouped together. In this transaction we’re referring to the database with the local variable `db`, telling the database that there should be a collection named `ideas` or creating one and starting it as an empty set. Then we shovel an idea with fake data into that collection of `ideas`.

```
$ irb
2.1.1 :001> require './idea'
=> true
2.1.1 :002> idea = Idea.new
=> #<Idea:0x007f86fc04a0a8>
2.1.1 :003> idea.save
=> NameError: uninitialized constant Idea::YAML
```
Loading `idea.rb` goes fine, but when we try to save, it blows up in `save` when it calls the `database` method because it doesn’t know what `YAML` is.

```
2.1.1 :001> require 'yaml'
2.1.1 :002> idea.save
=> NameError: uninitialized constant Psych::Store
```
The thing we’re using in the `database` method, `YAML::Store`, is a wrapper around another library named `Psych::Store`. We can pull it in by requiring ‘yaml/store’:

```
2.1.1 :001> require 'yaml/store'
2.1.1 :002> idea.save
=> {title: 'diet', description: 'pizza all the time'}
```

#### Finding the Form Data

Sinatra gives us a `params` method that returns an object holding the data we need.Let’s take a look at it by calling `inspect` and commenting out the rest of the `post '/'` block:

```
post '/' do
  params.inspect
  ## 1. Create an idea based on the form parameters
  # idea = Idea.new
  #
  ## 2. Store it
  # idea.save
  #
  ## 3. Send us back to the index page to see all ideas
  # "Creating an IDEA!"
end
```
Now go to localhost:4567 and fill in a new idea. Click submit, and the page shows you the following.

```
{"idea_title"=>"story", "idea_description"=>"a princess steals a spaceship"}
```
So we learn that the hash returned by `params` has keys `idea_title` and `idea_description`. Those names come the tags we defined in the HTML form.

#### Using the Form Data

```
#...

idea = Idea.new(params['idea_title'], params['idea_description'])

#...
```

#### Redirecting after save

The `redirect` method is provided by Sinatra. We give it a string parameter which is the path we want to redirect to. The redirect will come in as a `get` request.

#### Querying for the ideas

```
get '/' do
  erb :index, locals: {ideas: Idea.all}
end
```
*This means render the ERB template named `index` and define in that scope the local variable named `ideas` with the value `Idea.all`*.

#### Building Objects from a hash

The view template is trying to call the `title` method on what it gets back from `.all`. But `all` is returning a collection of hashes with what `YAML::Store` returns. That hash has a key `:title`, but not a method `.title`.

## I3: Deleting an Idea

1. Adding a unique idefntifier
2. Adding a delete button to index.erb
3. Defining the Delete route

##### Sinatra's `method_override`

Sinatra is still looking for a `POST` route, not a `DELETE`. Sinatra knows about the workaround using the `_method` parameter, but we need to enable it.

In your `app.rb` file add this line to the top of your class definition:

```
class IdeaBoxApp < Sinatra::Base
  set :method_override, true

  # ...
end
```

Now Sinatra will pretend that the incoming request used the `DELETE` verb instead of `POST`.

#### Add `delete` to Idea

```
class Idea
  def self.delete(position)
    database.transaction do
      database['ideas'].delete_at(position)
    end
  end
end
```

The `delete` method starts a transaction, accesses the `ideas` collection, then uses the `delete_at` method to remove the element at a certain position.

This is an example of "duck typing." The `delete_at` method in `YAML::Store` is built to work like the `delete_at` method on `Array`.

- HTTP requests are just made of strings. So when you use `params` in a Sinatra app, the values from the request are *always* strings. Our form is submitting the position as a parameter in the HTTP request. When we pass that parameter into the `delete` method the position is a string. But `delete_at` will only work with an integer. `id` needs to be an integer with the help of `.to_i`

## I4: Editing an Idea

1. Add an Edit Link
2. Add the Edit Action
3. Render an Edit Template
3.a. Create the Edit template in the *views* directory named `edit.erb`

```
<html>
  <head>
    <title>IdeaBox</title>
  </head>
  <body>
    <h1>IdeaBox</h1>

    <form action="/<%= id %>" method="POST">
      <p>
        Edit your Idea:
      </p>
      <input type="hidden" name="_method" value="PUT" />
      <input type='text' name='idea_title' value="<%= idea.title %>"/><br/>
      <textarea name='idea_description'><%= idea.description %></textarea><br/>
      <input type='submit'/>
    </form>
  </body>
</html>
```

Again we’re creating a form which uses the `POST` method, and giving Sinatra the extra information in `_method=PUT` to say that even though this is coming in with the `POST` verb, we actually want it to be a `PUT`

Then the form has `input` tags for the title and description. The `value` attribute uses the current values for the idea rather than having blank boxes.

#### Setting up the `id`

The `edit.erb` references `id` on line 8, but that local variable doesn’t exist. Jump back into the `app.rb` file and change the `get '/:id/edit'` method to specify a local variable named `id`:

```
get '/:id/edit' do |id|
  erb :edit, locals: {id: id}
end
```

The `locals: {id: id}` is saying "create a local variable for the ERB template with the name `id` which is a reference to the value in the `id` variable in this action’s scope."

## I5: Refactoring Idea

We've changed to make `Idea` take a hash by changing:
- `initialize` method
- class methods
- `post` action
- `index.erb`
- `edit.erb` and `update` action

##### Debugging a Type Issue

If you look at the database file, some of the ideas look like this:

```
---
ideas:
- :title: dinner
  :description: pizza
- :title: dessert
  :description: candy, soda, and ice cream
```

Whereas other ideas look like this:

```
---
ideas:
- title: dinner
  description: pizza
- title: dessert
  description: candy, soda, and ice cream
```

Essentially, `:title:` is the YAML for the Ruby symbol `:title`, whereas `title:` is YAML for the Ruby string `"title"`.

When the `params` object comes back, we send it directly to `Idea.update`. While we can access fields in the `params` using both strings and symbols for the keys, if we just grab the value of `params[:idea]`, we’ll get a hash with string values for the keys:

```
{"title" => "game", "description" => "tic tac toe"}
```

We can either jump through some hoops to deal with both strings and symbols, or change the update so we explicitly pass symbols to the database, or we can just use strings all the way through the app. Let’s just use strings.

We need to update the `initialize` and `save` methods in `idea.rb` to use strings for the hash keys instead of symbols

## I6: Using a View Layout

Sinatra is automatically noticing existing `layout.erb` and wrapping the `index.erb` with the layout.

However, to allow the content of the view template to actually be rendered, we need to add a call to yield inside the <body> tags like this:

```
<html>
  <head>
    <title>IdeaBox</title>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

If you view the HTML source in your browser, you’ll find that it has duplicated all the `<html>` and `<head>` tags. Open `index.erb` and `edit.erb` and delete all the wrapper which is in the layout. Now those view templates are just focused on what’s unique about that page.

## I9: Ranking and Sorting

#### Sorting Ideas

We want to be able to sort ideas, so let’s include the Ruby `Comparable` module in `Idea`:

```
class Idea
  include Comparable
  # ...
end
```
For `Comparable` to help us, we have to tell it how two ideas compare to each other. The way we do that is with the so-called spaceship operator, which looks like this: `<=>`.

```
def <=>(other)
  rank <=> other.rank
end
```

It turns out that our simplistic way of giving ideas an `id` based on their order in the list isn’t cutting it. The id in the index page is based on the sort order, whereas the id when we update an idea is based on the position of the idea in the database.
Let’s have the database tell the idea what its id is when it gets pulled out.



