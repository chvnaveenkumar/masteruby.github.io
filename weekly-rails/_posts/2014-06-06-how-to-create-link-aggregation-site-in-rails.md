---
layout: post
title: How to create link aggregation site in Rails&#58; Adding, editing, deleting links
---


In this post we'll create link aggregation that look like **Hacker News**.

Here's how it'll be look like:

![link aggregation site](/images/link_aggregation_site.png)

In this tutorial we'll cover:

* <a href="#design">Design our page with HTML, CSS</a>
* <a href="#devise">Creating authentication with Devise</a>
* <a href="#links">How to create, edit and delete links</a>
* <a href="#user">How to show link user and time when post was submitted.</a>

## <a name="design">Design our page</a>

Before we start to add real code, we'll start with html and css. First create application named **mlink**.

{% highlight sh %}
rails new mlink
{% endhighlight %}

We'll start with designing how our page will be look like. Add slim gem to your Gemfile

#### Gemfile
{% highlight ruby %}
gem 'slim-rails'
{% endhighlight %}

Slim is template language, that strips HTML from unecessary tags. 

Here's how look like code in HTML

{% highlight html %}
<html>
  <head>
    <title>My Webpage</title>
  </head>
  <body>
    <h1>My heading</h1>
    <p>My paragraph</p>
  </body>
</html>
{% endhighlight %}

And this how look like code in Slim:

{% highlight html %}
html
  head
    title My Webpage
  body
    h1 My Heading
    p My paragraph
{% endhighlight %}

Simpler isn't it? Slim strips HTML from all opening and closing tags. Slim uses two spaces indentation after every parent element. For example element **html** has two children **body** and **head**. In Slim you can use Ruby part of Ruby code.
    


### Creating navigation

We'll start with basic navigation, wewant to show things related to links on left side and authentication on right side. Create controller named home

{% highlight sh %}
rails generate controller home home
{% endhighlight %}

If you look at **app/views** folder you should see that we've just createdfolder with name **home** and you should see file **home.html.slim** inside it.

Let's take a look at the code at the browser:

{% highlight sh %}
rails server
{% endhighlight %} 

It works, but typing whole path to address bar in browser can be really disgusting. Let's fix it.

#### config/routes.rb

{% highlight ruby %}
root 'home#home'
{% endhighlight %}

Here we've changed root path. So if you look at this at the browser, you should see that we don't have to type address to addressbar anymore.

## Creating navigation

First thing we want to do is to create navigation, but we want to use it on every page so we need to modify our layout. If you have **application.html.erb** in your **layouts** folder replace it with following file:

#### app/views/layouts/application.html.slim

{% highlight html %}
doctype html
html
  head
    title Mlink
    = stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true
    = javascript_include_tag "application", "data-turbolinks-track" => true
    = csrf_meta_tags
  body
    div#wrapper
      = render 'layouts/header'
      = yield

{% endhighlight %}

If you scroll down to the code, you certainly noticed two methods, first is **render**. Render is used when you don't want to have all the code in one file and want to use it in other file. Method will look at the partial with name **_header.html.slim** in **layouts** folder.

Another method is **yield**. Yield is used to include anything related to page right after to header.

So now you know why to use partials, let's create partial **_header.html.slim**

#### app/views/layouts/_header.html.slim

{% highlight html %}
header.navbar
  ul.links
    li = link_to "Mlink, '#'
    li = link_to "Home", '#'
    li = link_to "Submit", '#'
  ul.auth
    li = link_to "Login", '#'
    li = link_to "Signup", '#'

{% endhighlight %}

Take look at the code that we've just done.

{% highlight html %}
header.navbar
  ul.links
{% endhighlight %}

Slim use shortcuts for creating html elements. So here, you've used this:

{% highlight html %}
header.navbar
{% endhighlight %}

instead of:

{% highlight html %}
<header class="navbar">
{% endhighlight %}

You certainly noticed we've used **link_to**. Link_to is special rails method for links.


Let's add some styling for our navigation. Create file **main.css.scss**

#### app/assets/stylesheets/main.css.scss

{% highlight css %}

.navbar {
  background-color: rgb(120, 163, 0);
  width: 100%;
  overflow: hidden;

  ul {
    list-style: none;
    
    li {
      display:inline;
      padding: 0.8em;
      
      a {
        text-decoration: none;
        color: white;
      } 
    }
  }
}

.links {
  float: left;
}

.auth {
  float: right;
}  
{% endhighlight %}

Here we've added green background to our navigation, add some place between elements and 

Ok let's run rails server to see whatwe've created.
We'll end up with menu like this:

![horizontal navigation](/images/header.png)

### Design main content of page

Let's design content our webpage. We want to show links that users submitted on homepage, so add this to file **home.html.slim**

#### app/views/home/home.html.slim
{% highlight html %}
section#content  
  ol#links-list	
    li
      span.link	
        = link_to "Geeklist", '#'
      span.details  
        p
          | posted by	
          = link_to "ed_wood", '#'
          | 20 minutes ago	

{% endhighlight %}

And styling to **main.css.scss**


#### app/assets/stylesheets/main.css.scss

{% highlight css %}
#wrapper {
  background-color: rgb(223, 223, 224);
  }

.details a {
  padding: 0.6em;
}

#links-list li {
  padding: 0.8em;
  a {
    color: rgb(120, 163, 0);
  }
}

{% endhighlight %}

So we've just added how links will be look like.

Your homepage should look like this:

![links](/images/links.png)

We've just designed our home page let's move to creating authentication.

## <a name="devise">Creating Authentication with Devise</a>

For authentication we'll use devise authentication system.

Add Devise gem to your Gemfile

{% highlight ruby %}
gem 'devise'
{% endhighlight %}

And install it.

{% highlight ruby %}
bundle install
{% endhighlight %}

Run following command to setup Devise

{% highlight sh %}
rails generate devise:install
{% endhighlight %}

To work, we need to add model **devise user** to database.

{% highlight sh %}
rails generate devise user
rake db:migrate
{% endhighlight %}

Now we have to add authentication links to our homepage. Add this code to **_header.html.slim**

#### app/views/layouts/_header.html.slim
{% highlight html %}
header.navbar
  ul.links
    li = link_to "Mlink", '#'
    li = link_to "Home", root_path
    li = link_to "Submit", '#'
  ul.auth
    - if user_signed_in?
      li = link_to 'Edit profile', edit_user_registration_path
      li = link_to 'Logout', destroy_user_session_path
    - else
      li = link_to 'Login', new_user_session_path
      li = link_to 'Signup', new_user_registration_path
{% endhighlight %}  

Restart your rails server and after you click on Signup link you should see this form:

![signup form in devise](/images/sign_up.png)

## <a name="links">Creating, editing, deleting links</a>

We've just added authentication. We need to create controller named **links**

{% highlight sh %}
rails generate controller links
{% endhighlight %}

We need to add links values to database, so we need to generate this model.

{% highlight sh %}
rails generate model Link title:string url:string user:references
rake db:migrate
{% endhighlight %}

Previous command will generate

Here we've created link attributes. Every link has title and url, and of course user id related to link url.

{% highlight sh %}
rake db:migrate
{% endhighlight %}



Add path do **config/routes.rb**

{% highlight ruby %}
resources :links
{% endhighlight %}

**Resources** will create all path

Now we can add path to controller. We'll add create, update and destroy.

#### app/controllers/links_controller.rb


{% highlight ruby %}
def new
end

def create
end

def update
end

def destroy
end
{% endhighlight %}

### Creating links

Now we'll create form for creating links. Add **/links/new** path to **_header.html.slim**

{% highlight html %}
- if user_signed_in
  li = link_to "Submit", 'links/new'
{% endhighlight %}

We have added user_signed which is helper that Devise provides to check if user has signed_in and if he has signed in we'll show him link to submit form.

Let's create form for our links, to simplify it add **simple_form** gem to your Gemfile.

#### Gemfile
{% highlight ruby %}
gem 'simple_form'

{% endhighlight %}

And configure it.

{% highlight sh %}
rails generate simple_form:install
{% endhighlight %}

Let's create form for submitting our links. We'll put form in a partial **_form.html.slim**


#### app/views/links/_form.html.slim

{% highlight html %}
= simple_form_for @link do |f|
  = f.input :title
  = f.input :url
  = f.button :submit
{% endhighlight %}

Simple form will generate basic html form.
 

#### app/views/links/new.html.slim
{% highlight html %}

h1 Submit a Link

= render 'form'
{% endhighlight %}

Just add little styling to your forms

#### app/assets/stylesheets/forms.css.scss

{% highlight css %}

.simple_form {
  div.input {
    margin-bottom: 30px;
    clear: both;
  }
  label {
    float: left;
    width: 200px;
    text-align: right;
    margin: 2px 10px;
  }
}
{% endhighlight %}

We need to connect user model with link model

#### app/models/user.rb

{% highlight ruby %}
has_many :links
{% endhighlight %}

And we validate in our model that our links form can't be blank.

#### app/models/link.rb

{% highlight ruby %}
belongs_to :user
validates :title, presence: true
validates :url, presence: true
{% endhighlight %}

#### app/controllers/links_controller.rb

{% highlight ruby %}
def create
  @user = current_user
  @link = @user.links.new(link_params)
  if @link.save
    redirect_to root_path
  else
    render 'new'
  end
end

  private
    def link_params
      params.require(:link).permit(:title, :url)
    end
end    
{% endhighlight %}



We want to our links to show on home page so add this code to home controller.

{% highlight ruby %}

def home
  @links = Link.all
{% endhighlight %}

And add some code to **home.html.slim**

{% highlight html %}
section#content
  ol#links-list
    - @links.each do |link|
      li
        span.link
          = link_to link.title, link.url
        span.details
          - if link.user == current_user
            = link_to 'edit', edit_link_path(link)
            = link_to "delete", link_path(link), method: :delete
{% endhighlight %}



### Editing links

We want to user to edit his links that he submitted, so we'll add update action to **links_controller**

#### app/controllers/links_controller.rb
{% highlight ruby %}
def edit
  @link = Link.find(params[:id])
end

def update
  @link = Link.find(params[:id])
  
  if @link.update(link_params)
    redirect_to root_path
  else
    render 'edit'
  end 
end 
{% endhighlight %}

Here we've find link by it's id and check out if link was submitted by current signed user, if it was we allow updating it.

### Deleting links

To delete links add destroy action to **links_controller**

{% highlight ruby %}
def destroy
  @link = Link.find(params[:id])
  if @link.user == current_user
    @link.destroy
    redirect_to root_path
  end
end
{% endhighlight %}

## <a name="user">Showing link's user and when it was submitted.</a>

We want to show which user submittted link and time when it was posted.

Right now we're using user's email to login, but we want to show username with link. We need to edit Devise to fix that.

### Adding username to authentication in Devise

To add username to Devise based authentication follow instruction on [devise wiki](https://github.com/plataformatec/devise/wiki/How-To:-Allow-users-to-sign-in-using-their-username-or-email-address)

If you followed instruction on Devise wiki, you can add link's user and time when it was submitted.

Add this code to **home.html.slim**

#### app/views/home/home.html.slim

{% highlight html %}

section#content
  ol#links-list
    - @links.each do |link|
      li
        span.link
          = link_to link.title, link.url
        span.details
          - if link.user == current_user
            = link_to "edit", edit_link_path(link)
            = link_to "delete", link_path(link), method: :delete

          p
            |
            posted by
            = link_to link.user.username
            = time_ago_in_words(link.created_at)

{% endhighlight %}



So that's all for now. Next time we'll add some comments to our links aggregation site.
