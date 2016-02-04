---
layout: post
title: "Ruby-fied Javascript Part II: Currying"
date: 2016-02-3 14:11:44 -0500
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

Chances are, you've actually seen a good amount of the function .bind(), it's a very common way to deal with the "this" problem in Javascript. In short, .bind() allows you to dictate what this" will refer to inside the function at the time the function is called. From this property alone, you can probably guess how this will allow us to start borrowing functions.

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
The last two methods we are going to go over are .call() and .apply().

Truth be told these functions are very similar to .bind() but there are a few key differences. Let's start with the differences between .bind() and .call().

```Javascript
function saySomething(name){
  return this.greeting + name + ", you old dog you!"
}

var hello = {
  greeting: "HALLO "
}

var supChip = saySomething.bind(hello, "Chip")

saySomething.call(hello, "Duckie") //-> "HALLO Duckie, you old dog you!"
supChip() //-> "HALLO chip, you old dog you!"
```

Here we begin to see the difference between these two very similar methods. They both set this with their first argument and set subsequent arguments in the method with their next parameters. However, .bind() returns a new function, whereas .call() immediately evaluates the function it's called on with it's special parameters.

.apply() is more similar to .call() than .bind(). In fact, the only difference is in the way .apply() is called.

```Javascript
function saySomethingElse(name, color, location){
  return this.introduction + name + ". My favorite color is " + color + " and I'm from " + location
}

var intro = {
  introduction: "YOU GUYZ I'M "
}

saySomethingElse.call(intro, "Adam", "Aubergine", "Austria") //-> YOU GUYS I'M Adam. My favorite color is Aubergine and I'm from Austria
saySomethingElse.apply(intro, ["Benny","Burgundy","Belize"]) //-> YOU GUYS I'M Benny. My favorite color is Burgundy and I'm from Belize
```
As you can see, the arguments passed through .apply() must be passed in an array. This is useful for functions where the arguments being passed in are more flexible or optional.

##A Ruby in the Rough
![Ruby is rough](http://kardsunlimited.com/wp-content/uploads/2013/06/tumblr_lounw0Da7r1qi6sy1o1_500.png)

As we've spoken about before, Ruby can play functional with the big dogs, and as it's evolved as a language, it's gained even more functional properties. We're going to explore a few ways of replicating this awesome functionality from Javascript.

####Waddya Mean Modules?!
![What if I told you?](http://blog.wassill.eu/sites/default/files/styles/adaptive/public/sfkuxr4.png?itok=FUXr3pHL)

The first method of sharing methods between classes is one we are all pretty familiar with. A Module is a different type of object that can mix methods into another class. Much like binding this to a function, the Module and Class need to share certain functionality to begin with.

```ruby
module Greetable
  def say_hi
    "HI #{self.name}"
  end

  def say_bye
    "BYE #{self.name}"
  end
end
```

As long as a class has a name attribute, it can easily become "Greetable" by using the following code:

```ruby
class Person
  attr_accessor :name

  include Greetable
end
```

Now person has the methods #say_hi and #say_bye!

I won't delve too far into Module functionality, since it's pretty well covered, but this is a peek at how Ruby can share methods between classes. Much like Javascript leverages the "this" keyword to allow for code that is applicable in many scenarios, Ruby's "self" keyword affords similar functionality in it's Object Oriented design.

####Call me Ruby?
![This is dog](http://weknowmemes.com/wp-content/uploads/2012/05/hello-maybe-this-is-dog.jpg)

Turns out that Ruby has it's own version of .call() and unsurprisingly it's strongly related to the Proc class. Any proc accepts a message of .call as a command to run the code inside the proc, so simply put, it's a way of running a proc. Given that Procs don't arity check (unless they are lambdas), this can be used to replicate some of the usefulness of .call() or .apply(). However, as of Ruby 1.9.3 there is one  more useful function.
####Keep Calm and Curry On
![Curry Puns LOLZ](http://bircurries.co.uk/forum/download/file.php?id=1067&t=1)

Ruby gained full partial function application when it added the .curry function. In 1.9.3 .curry could be called on any proc or lambda and it would return a new proc or lambda that had one of it's arguments set with a specific parameter. If this functionality is still a bit fuzzy, ponder the following code.

```ruby
do_math = lambda do |operator, x, y|
  a.send(operator, b)
end

do_math.(:+, 2, 3) #-> 5
do_math.(:*, 2, 3) #-> 6
do_math.(:-, 3, 2) #-> 1
```
This bit of metaprogramming is pretty straightforward, but we can break this down a bit further using #curry.

```ruby
add = do_math.curry(:+)
add.(2,3) #-> 5
add.(10,5) #-> 15

subtract = do_math.curry(:-)
subtract.(4,2) #-> 2
subtract.(12,9) #-> 3
```
When we called #curry on #do_math we set the first argument (the operator) and store the resulting function in a new variable. We can even take this a step further.

```ruby
increment = add.curry(1)
increment.(100) #-> 101
```

As you can see, the potential here to dry up code is enormous. If one design their functions properly, one function is powerful enough to write plenty. However, the key work is "design", it takes a lot of planning or refactoring in order to achieve this next level DRYness.

![So Dry](http://img.pandawhale.com/post-58159-Im-gonna-make-it-so-dry-for-yo-6Yyj.gif)

####Final Note
As of Ruby 2.2.0 there is now a way to call curry on any method or block (instead of just on procs and lambdas). It looks something like this:

```ruby
def say_hi(name)
  "Hi #{name}"
end

chet = method(:say_hi).curry("Chet")
chet.() #-> Hi Chet
```
If you'd like to read more, check out the [documentation on 2.2.0's method class](http://ruby-doc.org/core-2.2.0/Method.html) - there is lot's of other cool functional aspects there.


##Resources
You know, because god knows I didn't learn this on my own.

- [Javascript is sexy](http://javascriptissexy.com/javascript-apply-call-and-bind-methods-are-essential-for-javascript-professionals/)
- [Stackoverflow Answer](http://stackoverflow.com/questions/15677738/whats-the-difference-between-call-apply-and-bind)
- [Good blog on apply and bind](http://hangar.runway7.net/javascript/difference-call-apply)
- [How I even learned about Currying](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-ii/)

Lastly, RUBY CURRY
![Ruby Curry!](http://ramblingsofafoodaddict.com/wp-content/uploads/2014/05/Chicken-Ruby.jpg)
