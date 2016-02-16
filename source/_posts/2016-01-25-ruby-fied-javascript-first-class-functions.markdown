---
layout: post
title: "Ruby-fied Javascript Part I: First Class Functions"
date: 2016-01-25 14:20:12 -0500
comments: true
categories: "Flatiron School"
---
As I've started learning Javascript, I've had alternating moments of progressively confusing rage, soul-encompassing ["wat"](http://knowyourmeme.com/memes/wat), laced with moments of being genuinely impressed by Javascript's functional elements. It's like eating your least favorite ice cream flavor mixed with your favorite sprinkles.

!["What is this, I don't Even"](http://i1.kym-cdn.com/photos/images/original/000/517/320/71c.gif)

However, as I've delved deeper into some of these features, I've been inspired to go back to Ruby and see which of these features have equivalents, and I've been pleasantly surprised to see that more than a few do.

In this first part, I want to explore how Ruby handles First-class Functions.

## What is a First-class Function?
When a programming language is said to have First-class Functions, it's a way of saying that the language allows for functions to be treated like objects, and passed back and forth as arguments.

So when we pass in a function as an argument in JS, or even get one back as a return value, we are exploiting this incredibly useful function of Javascript. As seen below:

```Javascript
function theOneWePass(){
  return "This is from the function passed in";
}

function theOneWeCall(theOneWePass){
  console.log(theOneWePass());
  return function(){
    return "and this is the function that was passed back";
  };
}

var thisHoldsAFunction = theOneWeCall(theOneWePass); // logs "This is from the function passed in"
thisHoldsAFunction(); //->"and this is the function that was passed back"
```

Strictly speaking, ruby does not support First-class Functions. Methods cannot be return values or held in variables. But methods are far from all Ruby uses to implement functions. And some of these functions have Javascript-y super powers.

##Blocks and Yield
Nearly anyone who has touched Ruby is probably familiar with a block. It allows us to customize methods by passing in custom instructions that are executed somewhere within the method, using the **yield** keyword.

```ruby
def some_method(array)
  array.each do |elem|
    yield(elem)
  end
end

some_method([1,2,3,4]) do |elem|
  puts (elem * 2)
end #-> 2, 4, 6, 8
```

In this example we are actually passing a block into a method twice, first when we call #some_method and next when we call the #each method. This is almost like passing a function as an argument, except we define the whole function, and instead of calling it by name, it gets called when we use the **yield** keyword. It's pretty much a regular closure and it even has the coolest property of a closure, which is access to the scope in which it was defined.

```ruby
x= 10

some_method([1,2,3,4]) do |elem|
  puts (elem * x)
end #-> 10, 20, 30, 40
```

But there is one important difference, we have to write out the function, we still are not assigning it to a variable.

However, there is a way we can do that!

## Procs! Procs Everywhere!
![Procs Everywhere](http://i.imgur.com/wJ0cIkO.jpg)

A Proc is how ruby wraps a function into an object, and best of all, they can be defined and saved to variables! This means that they can be called at will with the .call method.

```ruby
  def gen_exponential(factor)
    return Proc.new do |n|
      sum = n
      (factor-1).times do
        sum *= n
      end
      sum
    end
  end

  cuber = gen_exponential(3)
  cuber.call(3) #-> 27
  cuber.call(4) #-> 64
```

You can even pass that Proc back and forth into functions.

```ruby
  [1,2,3,4].map do |elem|
    cuber.call(elem)
  end #-> [1,8,27,64]

  def cool_map(array, something_to_call)
    new_array = []
    array.each do |elem|
      new_array << something_to_call.call(elem)
    end
    new_array
  end

  cool_map([1,2,3,4], cuber)#-> [1,8,27,64]
```

So far so cool right?

So quick recap, both blocks and procs allow you to pass code into other methods. BUT, procs are objects, and blocks are not. Procs can be passed into methods as arguments, and can be called with the .call(), and blocks can only be passed by writing the whole block out, they must be called with the yield function, and each method can only have one block.

Now let's go deeper

## How Many Procs could a Proc Block Proc if we use the Unary Operator?

![BRING IT](http://i2.photobucket.com/albums/y42/GilGrissomCSI/woodchuck.jpg)

Let's take a look at this piece of code.

```ruby
def some_method(&block)
  block.class
end

some_method do

end #-> Proc
```

The first thing you should know, is that the ampersand is also known as the unary operator, and it's a means of turning Blocks into Procs and visa versa. In the above example, it is used to turn the block into a proc, i.e. make it an object. Once it's an object, we can call object methods on it, and see that it is actually a proc. The unary also works in the opposite direction, turning procs into blocks which can then be passed into methods that need a block.

```ruby
putstuff = Proc.new{|thing| puts thing}

[1,2,3,4].each &putstuff #-> 1,2,3,4
```

This allows us to save methods in variables and pass them into blocks with relative ease.  But there is one more step to consider when looking at how

##Mary had a little Lambda
![Soon](http://3.bp.blogspot.com/-8AfGm4Uy3CU/UBqhxWvD7VI/AAAAAAAAARU/iVQAHfgv3cs/s1600/a-wolf-in-sheeps-clo_1339994049_epiclolcom.png)

Now there is one more way in which Ruby supports first class functions: lambdas. Lambdas are actually a lot like procs, but almost a different "flavor". Lambdas can be passed back and forth, and are called similarly to procs, but they are declared slightly differently, and have a few other important differences.

```ruby
proc_town = Proc.new {}
lambda_ville = lambda {}

#they are the same class
proc_town.class #-> Proc
lambda_ville.class #-> Proc

#but if you look closer
proc_town #-> <Proc:0x007ff4b3d47510@(irb):59>
lambda_ville #-> <Proc:0x007ff4b3d37660@(irb):60 (lambda)>
```
There are two other key differences, which very much affect the way these are used.

#####Arity Checking
As we've seen with Javascript, sometimes languages do not care about whether the number of arguments you pass to a function match the number of arguments that function takes.

Similar to Javascript functions, Procs will not throw an argument error if they are passed a varying number of arguments, while Lambdas do enforce argument strictness (or in other words, they arity check). This makes procs a little dangerous, only in the sense that they will fail silently if you mess something up (like Javascript), but they are also more flexible.

#####Return of the Lambda
Procs and Lambdas differ in the way that they return values to where they are called. A Proc does not create it's own "return scope", meaning it almost becomes a part of the function that calls it. So if a Proc triggers the return keyword, it will short-circuit the rest of the method that called it.

```ruby
def messing_with_procs
  foo = Proc.new {return "'nothing' is wrong!"}
  foo.call
  puts "Sometimes Proc's fail silently"
end

puts messing_with_procs #-> "'nothing' is wrong!"
```

Lambdas are different, they do create their own return scope, so if a lambda triggers the return keyword, it will exit the lambda code, but return to the method that called it.

```ruby
def sweet_sweet_lambdas
  bar = lambda {return "Actually nothing is wrong"}
  puts bar.call + "because lambdas are so so sweet"
end

puts sweet_sweet_lambdas #-> "Actually nothing is wrong because lambdas are so so sweet"
```

##Recap
So let's recap, one of the cooler things about Javascript is that it supports First-class functions, which means passing functions to functions and having them return other functions (some of which also take and return functions). FUNCTIONS. Being passed back and forth. Forever.
![Yo Dawg](http://i.imgur.com/e7RHh2a.jpg)

Turns out, ruby ain't no slouch. Though the traditional method of passing code to functions through blocks is pretty limited, we still have the options of procs and lambdas, which give ruby true support for First-class functions.

## Resources

I took a lot of things from people smarter than I, here they are:
[Functional Programming Techniques With Ruby: Part II](http://www.sitepoint.com/functional-programming-techniques-with-ruby-part-ii/)

[JavaScript’s Apply, Call, and Bind Methods are Essential for JavaScript Professionals](http://javascriptissexy.com/javascript-apply-call-and-bind-methods-are-essential-for-javascript-professionals/)

[Ruby Monk on Blocks Procs and Lambdas](https://rubymonk.com/learning/books/4-ruby-primer-ascent/chapters/18-blocks/lessons/64-blocks-procs-lambdas)

[What Is the Difference Between a Block, a Proc, and a Lambda in Ruby? (I actually think this is a former flatiron blog)](http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/)
