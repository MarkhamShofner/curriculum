# Devise! and sessions

## Learning Objectives
- explain how sessions give state to a web application
- explain how user authentication utilizes sessions
- set session hash key value pairs inside of a rails application
- implement user authentication into a web application utilizing the devise gem
- implement useful helper methods devise provides

### Opening Framing (10m)
- So we've figured out how to execute CRUD functionality into our apps. This is great, but should anyone using our application be able to do whatever they want? It would really suck if your buddies could post for you on facebook or if someone withdrew money from your banking site.

### T & T
Well, that shouldn't be the case. For the next 5 minutes turn to your partner in class and discuss how we might be able to prevent people from doing whatever they want on your website.

Here's some talking points:
- What does it mean for a user to be signed in?
- For that matter, what does it mean to be a user? How are we encapsulating that data.
- We've used our backend to store data about apartments and tenants. Can we use it to store data about a user itself?

> We need to start thinking about how to capture information into a User model. That has information about username, email, or password. We also need a way to transfer or perserve state in our applications from request to request.


## Enter Sessions(i do, talk + setup)15m
- HTTP is stateless, so it's up to either the browser or application that needs to "remember" what's happening
- A session is just a place to store data during one request that you can read during later requests.
- A session is a hash containing key value pairs that provide state in your application

Let's hop into some code so we can see sessions in action.

The code we'll be working with today can be found [here](https://github.com/ga-dc/tunr_rails_sessions_devise) This is a version of the tunr rails application we've been building on this week. It contains everything that we've learned up until now.

Make sure you fork/clone this repo if you would like to follow along. It is not too important to follow along for this portion of this lesson. I encourage you to just watch and use the following code to practice it yourself later.

## Set and Use session key-value pairs

In order to understand sessions, we're going to add some code to our existing artists controller. In `app/controllers/artists_controller.rb`:

```ruby
# this action will set `test: "this string was set on the testing_session action"` as a key value pair in the session
def testing_session
  session[:test] = "this string was set on the testing_session action"
end

# also add @test = session[:test] to index action
def index
  @test = session[:test]
  @artists = Artist.all
end
```

Let's update our `app/views/artists/index.html.erb` to include the use of our new instance variable `@test`. Place the following code somewhere in that file:

```html
<h1>The value of @test is: <%= @test %></h1>
```

Now because we created this action, let's update our `config/routes.rb` to listen for a request to that action:

```ruby
get '/artists/testing_session', to: 'artists#testing_session'
```

Finally we need to create a view for this action. Lets create that now `$ touch views/artists/testing_session.html.erb`. Let's make this view simple and just put in the following content.

```html
<h1>A session key value pair is being set</h1>
```

Let's first navigate to our artists index route at `http://localhost:3000/artists`.

We can see that there is nothing after `The value of @test is: `

Now let's navigate to `http://localhost:3000/artists/testing_session`

Great that's nice, our view was generated, but more importantly we had a key-value pair get set in this request because of the `testing_session` action in our artists controller:

```ruby
def testing_session
  session[:test] = "this was set on the testing_session action"
end
```

How do we verify this session key-value pair was set? Let's navigate back to artists index path.

Now we can see `The value of @test is: this was set on the testing_session action`

If we open a new incognito window and navigate to `http://localhost:3000/artists`, we'll see the session isn't set in that browser because it's a brand new session.

When we clear our cookies/session in our browser we can see in our index route that `@test` becomes nil again.

> Now that we have the ability to transfer information from request to request. We can preserve state in our application.

## Setting objects in sessions
Instead of hardcoding strings into our sessions hash, lets see if we can store objects themselves in the session.

Let's update our show action to set a session. In `app/controllers/artists_controller.rb`:

```ruby
def index
  @last_viewed_artist = session[:last_viewed_artist]
  @artists = Artist.all
end

def show
  @artist = Artist.find(params[:id])
  session[:last_viewed_artist] = @artist
end
```

In your `app/views/artists/index.html.erb place the following:

```html
<h3>Last viewed artist: <%= @artist["name"] %></h3>
```

Now every time We click on an artists show page it will update the session to contain the most recently viewed artist. Then any time we make a request to the artists index path we can see the most recently viewed artist's name.

> So now we can store objects or attributes of objects in our sessions as well. This may include id's of a user model, which is basically what devise is going to do for us when a user "signs in"

## Deleting parts of the session
Lets add a new action/route/view for deleting session.

In `app/controllers/artists_controller.rb`:

```ruby
def deleting_session
  session.delete(:last_viewed_artist)
end
```

In `config/routes.rb`:

```ruby
get '/artists/deleting_session', to: 'artists#deleting_session'
```

Lets create `app/views/artists/deleting_session.html.erb` and add the following content:

```html
<h1>deleting the last viewed artist session key value pair</h1>
```

When we navigate from the `deleting_session` route back to the artist we can see an error of undefined method.

> This is the sort of thing thats happening when a user "logs out"

Checkout the [testing_sessions branch](https://github.com/ga-dc/tunr_rails_sessions_devise/tree/testing_sessions) for all the code thus far.

### You do
Take the next 5 minutes to:
- create a new action in the posts controller that instantiates a session key-value pair
- save it to an instance variable in the index, and have it display in the index view

### Break 10m
## Reframing
Devise is a gem in rails that we use in order to streamline user authentication. We're going to talk about how to get devise up and running and a couple of helper methods.

If you'd like to learn about hand rolled user authentication reference [this lesson plan](https://github.com/ga-dc/curriculum/tree/master/05-mvc-with-rails/rails-sessions-and-auth)

## Devise We do codealong (50m)
- Fork and clone [this repo](https://github.com/ga-dc/devise_blog/tree/starter)
- This repo contains a rails application that is a single model crud application for posts.
- the first thing we should do is add `devise` to the Gemfile and run a bundle install

```ruby
gem devise
```

Next install devise then generate the devise User

```bash
# normal bundle install
bundle install

# installs devise
rails generate devise:install

# generator used to create devise user model
rails generate devise User
```
You may have to restart your server at this point.

Alot of code just got generated for us.

This is some of the stuff that it's given us:

![railsgdeviseuser](images/railsgdeviseuser.png)

> Take a look at the devise source code! and you can see all of the different controllers

Its alot but of stuff to look at, but let's break it down. Going from top to bottom excluding some of the files we won't be using for this class.

The first thing that was created was a migration for this model. In `db/migrate/<somedate>_devise_create_users.rb` there's alot of information here, but I think most pertinent to us in the scope of user auth is the email and password attributes.

The next thing that I see is a model defintion was created for us in `app/model/user.rb`:

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end

```

> the different arguments being passed in to devise are very semantically named as far as their utilities go.

The next thing that was added was `devise_for :users` in our `config/routes.rb`

That one line of code opens routes to alot of devise user authentication controller actions

## Devise -configuration(15/45)
add this to `config/environments/development.rb`:

```ruby
 config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
 ```
> In an email there is no context for host and port where as other urls are relative and don't need explicit host and port

 I think it'd be nice we just went ahead and added a logout button to the layout so that we can easily log in and log out as a user. So in our `app/views/layouts/application.html.erb` add the following code:


 ```html
<% if current_user %>
  <h3><%= link_to 'Signout', destroy_user_session_path, :method => :delete %></h3>
<% else %>
  <h3><%= link_to 'Signup', new_user_registration_path %></h3>
  <h3><%= link_to 'Login', new_user_session_path %></h3>
<% end %>
```

> current_user is a helper method that allows us to access the user if one is logged in. If they are logged in, the signout link will appear, if not both the signup and login link will appear.


Let's also add a root path to our `config/routes.rb`:

```ruby
root 'posts#index'
```

## Devise Helpers (15/60)

Let's go ahead and fire up our server `$rails s` and sign up for our site!

Welcome aboard! You're an authenticated rider on rails! What does that really mean though?

(ST-WG) What's fundamentally different about our application than before? ..

So we've signed up and can see that nothing really has changed, we can still access our posts like normal.

Let's add a bit of code too see if we can garner some more functionality.

The first thing that I want to do is add the helper method `authenticate_user!` in the index action of the posts controller. So inside `app/controllers/posts_controller.rb`:

```ruby
def index
  authenticate_user!
  @posts = Post.all
end
```

You'll notice now, if i'm not logged in, i can no longer access the index action and it redirects me to the signup page if i try to access this action. This is really cool. I now have a way to restrict controller actions unless people are logged in.

This sort of thing is common with alot of actions or all actions of a controller. So we can use a `before_action` callback to handle this. Let's get rid of the `authenticate_user!` in the index action and put the following code at the top of `app/controllers/posts_controller.rb`:

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!, only: [:create, :edit, :update, :destroy]
  def index
     # ... more code follows
```

> note I only want to authenticate the user for the create edit update and destroy actions, it doesn't matter to me who is able to view posts. Though this is totally dependent upon the domain model.

We saw `current_user` above earlier. But we only took advantage of its truthiness/falsiness to generate some links for us. I think more importantly though, `current_user` is a helper method that returns a ruby object in our database that represents the user that is logged in.

To illustrate this more effectively. Let's add a bit of functionality. What we want to achieve is for users to be able to have many posts and for posts to belong to a user.

Let's do the following:
- add `has_many` and `belongs_to` associations in our model definitions
- add a foreign key to our posts table /w migration (`rails g migration add_foreign_key_to_posts user_id:integer`)
- `$ rake db:migrate`
- in the index action, change `@posts = Post.all` to `@posts = current_user.posts`

> note that since we don't have the `authenticate_user!` helper on the index action, if we were to access posts directly without authenticating it would error out. So to fix that we should enter some if/else conditional to handle this.

Something like this:

```ruby
def index
  if current_user
    @posts = current_user.posts
  else
    @posts = Post.all
  end
end

```

- in the create action , change `@post = Post.new(post_params)` to `@post = current_user.posts.build(post_params)`

> So we aren't able use new on the posts helper method. `.build` is the equivalent of `.new`. `.create` is the the same when using has_many helper getters. Additionally when we `.build` or `.create` we don't have to pass in the foreign_key or object in as an argument because we're "building" or "creating" off of that object.

## Class Ex
- Add devise to tunr!

## What you can look forward to with devise! (Should you want to learn more on your own, or if we have extra time I can talk about devise views a little bit.)

[Devise Documentation](https://github.com/plataformatec/devise)

This documentation contains alot of great information.

- Customize devise views
- Customize devise model attributes [Direct Link to Docs](https://github.com/plataformatec/devise#strong-parameters)

If you want to add devise to a brand new rails application check out this [blog post](http://andrewsunglaekim.github.io/Getting-a-handle-on-devise/)