# Partials in Rails

## Objectives

1. Proper use of partials
2. Using partials with collections of objects
3. Pass local variables into partials
4. Partial conventions in Rails

## Proper use of partials

Before we can get into how to use partials properly we first need to know what a partial is in Rails and what kind of behavior it can give an application. Essentially a partial is a snippet of view code that can be utilized on multiple pages. From a best practice perspective, there are a number of situations where an application should utilize partials, below are a few of them:

* Whenever a snippet of view code is utilized in multiple files. Remember DRY (Don't Repeat Yourself)? Partials make it possible to call a single file instead of having duplicate code throughout an application.
* Whenever a view file is getting too full and needs to be organized.
* To clean up how collections are rendered (more on that in the next section)

A basic example of how to use partials is built directly into the Rails scaffolding generator (not that I'm recommending using scaffolds since that can make the TDD process a little clunky). If you run a command such as:

```rails g scaffold Post title:string description:text```

It will give you a view directory that looks like this:

views/
> _form.html.erb
> edit.html.erb
> index.html.erb
> index.json.jbuilder
> new.html.erb
> show.html.erb
> show.json.jbuilder

The partial in this directory is the ```_form.html.erb``` file. Placing an underscore before the filename is how you tell Rails that the file will be a partial. So what makes the ```form``` file a good partial? If you open the ```new``` and ```edit``` files you can see that they do not actually have the ```new``` or ```edit``` forms, instead they simply call the ```form``` partial with the Ruby call ```<%= render 'form' %>```. Imagine a situation where you did not utilize partials for your forms, the result would be that anytime you added or removed a database column you would have to edit two files instead of one, which not only would create more work but could also lead to bugs if... scratch that... when you would forget to update one of the files.

What are other good candidates for leveraging partials? Below are a few you will come across on a regular basis:

* Navigation bars
* Footers
* Tables that have the same instance variables
* And any view code that is used twice in your application

## Using partials with collections of objects

Not only are partials good for DRYing up your view code, they can also be utilized to clean up how collections are called. Let's pretend that you have a news application that rendered all of the users that are following an article. Without partials your view code would look something like this:

```
<% @article.followers.each do |follower| %>
  <div class="article-followers">
    <%= image_tag(follower.avatar.to_s) %>
    <span class="article-follower-link"><%= link_to follower.name, user_path(follower) %></span>
    <span class="article-follower-location"><%= follower.location %></span>
    <span class="article-follower-activity"><%= follower.total_comments %></span>
  </div>
<% end %>
```

You get the picture, it isn't pretty, that's where partials come in handy, one way to use partials would be to replace that code with something like this (don't worry about the local call, we'll get into that in the next section):

```
<% @article.followers.each do |follower| %>
  <%= render partial: 'followers/follower', locals: { follower: follower } %>
<% end %>
```

Much better, right? You would simply move that ```div``` into the ```followers/follower``` partial and the view would render the same results. And as nice as that looks, Rails is all about clean code, which is why the entire collection can actually be refactored into a single line, such as the one below:

```
<%= render @article.followers %>
```

That one line of code will help make your view code much more manageable and will result in the same collection being generated for the end user. How exactly does Rails know where to find your partial? This falls in line with the convention over configuration paradigm. Put simply, Rails model's have a built in method called ```to_partial_path``` that has a default setting of the model name as a directory and the model as the partial. For our example above, if you ran ```Follower.new.to_partial_path``` in the console, you'd get ```'followers/follower'```. Pretty cool, right?

## How to pass local variables into partials

By default, instance variables are available to partials, this makes sense because the partials are simply view code snippets that are being called and 'slide' into the view when the file is being rendered. However there are times when you need to pass local variables to a partial and Rails a clean way to do this. We already saw this in the example from the previous section:

```
<% @article.followers.each do |follower| %>
  <%= render partial: 'followers/follower', locals: { follower: follower } %>
<% end %>
```

This will pass in the ```follower``` local variable to the partial so it will be available and can be called like any other local variable.

In addition to passing a local variable, you can also hard code a value, such as below:

```
<% @article.followers.each do |follower| %>
  <%= render partial: 'followers/follower', locals: { follower: follower, avatar_size: 42 } %>
<% end %>
```
This can be helpful if you want to use a partial in multiple places, but you need to dynamically pass in values based on what page is making the call to the partial. For example, if you have one page that you want to see large avatar images for the followers and another page that you want a small thumbnail, this syntax will let you control the value efficiently.

## Partial conventions in Rails

So far we have looked at standard partial conventions, such as the ```form``` being placed in the same directory as the files that are calling it, and then we have reviewed how models have built in methods where they look for partials with a specific naming structure. However how do you call partials that are not associated with a specific controller or model?

Let's say that we want to control where alerts are called, instead of placing them in the ```layouts/application.html.erb``` file (which you should never do anyways), you could create a new directory in the view's directory called ```shared``` and then add in a file called ```_alerts.html.erb```, now you can call your alers from anywhere in the application by calling:

```<%= render "shared/alerts" %>```

This same principle could be applied to any view code snippets that are used throughout the application, but are not directlry associated with a specific controller or model.

## Summary

To review, partials in Rails are a powerful tool that will let you DRY up your code and ensure that you are organizing your application view code properly.