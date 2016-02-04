---
layout: post
title: "Ruby-fied Javascript Part II: Currying"
date: 2016-01-26 14:11:44 -0500
comments: true
categories: "Flatiron School"
---

This is the Electric Boogaloo in my series on discovering the functional and sexy aspects of Javascript within Ruby. For those of you who haven't read [Part I](http://aemrox.github.io/blog/2016/01/25/ruby-fied-javascript-first-class-functions/) of this series, you should check it out, because I'll be relying on a few of those concepts in the following blog.

Today, I want to explore the concept of currying, and borrowing code from other functions (or even entire functions) to write DRYer, more compact and elegant code. Brace yourselves:

![Next Level Shit](http://i.imgur.com/SKwuFrg.jpg)

##Javascript - All Types of Curry

Before we delve into the depths of ruby madness, let's wrap our heads around the concept of what it exactly means to borrow functions, and how it works in Javascript.

Javascript has three separate methods that all allow you to bend functions to your will (and to your specific purpose): Bind, Apply and Call. They each have their own limitations and purpose, so let's go one by one and pick apart how each of these methods work.

####The Ties that Bind()
![bad bondage pun](https://s-media-cache-ak0.pinimg.com/736x/d9/83/28/d983287bb6e589eea0511f1c09288ef0.jpg)

Chances are, you've actually seen a good amount of the function .bind(), it's a very common way to deal with the "this" problem in Javascript. In short, .bind() allows you to dictate what "this" will refer to inside the function at the time the function is called. From this property alone, you can probably guess how this will allow us to start borrowing functions.

When functions are written vaguely enough, operating on objects using primarily the "this" keyword, they become fairly easy to bend. Let's take a look at an example:

```Javascript
function Widgets(name, quantity, price) {
  this.name = name;
  this.quantity = quantity;
  this.price = price;
  this.costOfInventory = function(){
    var totalCost = this.quantity * this.price
    return "The cost of the entire inventory of " + this.name + " is $" + totalCost
  }
}
```

In the function constructor we just defined, we managed to design the function costOfInventory so that it does not take any arguments, and only operates on functions attached to an object, referred to as this. Now we can use this function to refer to any object that has similar structure. For instance:

```Javascript
var wonkyWicket = {
  name: "Wonky Wickets",
  quantity: 100
  price: 10
}

Widgets.costOfInventory().bind(wonkyWicket) // -> "The cost of the entire inventory of Wonky Wickets is $1000
```
This gets even more powerful when you consider Javascript's support for [first class functions](http://aemrox.github.io/blog/2016/01/25/ruby-fied-javascript-first-class-functions/):

```Javascript
wonkyWicket.costOfInventory = Widgets.costOfInventory().bind(wonkyWicket);
```

now wonkyWicket has effectively borrowed the costOfInventory function from the Widget constructor, and it can be called normally. BUT WAIT, there's more! Bind can take additional arguments, and these arguments set default values for the arguments in function .bind() is being called on.

```Javascript
var fakeItem = {
  quantity: 100
  price: 10
}

Widgets.costOfInventory().bind(fakeItem, "Something AWESOME") // -> "The cost of the entire inventory of Something AWESOME is $1000
```

What the bind function is actually doing is creating a new function with default values in the place of the arguments that you defined in bind. This process is known as currying or partial function application. You substitute hard values for arguments, thus modifying the function.

Nuts right? And it gets even better when we start dealing with useful things like arrays and strings.

One important thing to note that will persist with us is that it's not as simple as calling .bind(), there is a decent amount of prep involved. Being able to borrow or curry functions is something you have to think through from the beginning, almost like how a certain design principles have some upfront work, but make your life easier down the road.


####I'm Call()in' you Out, Apply() Yourself!
The last two methods we are going to go over are .call() and .apply()


##A Ruby in the Rough
![Ruby is rough](http://kardsunlimited.com/wp-content/uploads/2013/06/tumblr_lounw0Da7r1qi6sy1o1_500.png)

####Waddya Mean Modules?!

####Keep Calm and Curry On
![Curry Puns LOLZ](http://bircurries.co.uk/forum/download/file.php?id=1067&t=1)
