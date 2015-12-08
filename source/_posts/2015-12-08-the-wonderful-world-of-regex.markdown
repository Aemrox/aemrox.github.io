---
layout: post
title: "The Wonderful World of Regex"
date: 2015-12-08 13:52:56 -0500
comments: true
categories: "Flatiron School"
---

While working on my last project, our main goal was solving one very difficult problem.

While ingesting around 1,000 articles a day, we needed to add some semblance of location data to each one of those articles. When we first launched, this was done by me. Manually. All. Goddamn. Day.

I would manually scan the article for some kind of address, or area (like 5th ave between 42nd and 43rd). We noticed that I was following very similar patterns. Addresses are generally clean patterns, and areas often use the same prepositions over and over again. My partner figured that we could begin parsing these texts automatically with a series of rules.

Since I was not any kind of developer at the time, I was less involved with the coding of the solution, but one way I was able to chip in was in writing the logic of finding these phrases through something called Regex.

Regex stands for Regular Expressions, and is a very powerful tool used for identifying, matching and capturing certain patterns in text and strings.

We would write regular expressions to begin automatically filtering and flagging the articles we were pulling in for items that didn’t fit what we were doing, like sponsored advertising:

`/((East|West)? ?([0-9]+|[A-Z][a-z]+|first|second|third|[a-z]+th)(st|th|nd|rd))/`

But more importantly, we wrote a series of expressions that would look for the prepositions connecting areas, or the patterns of addresses. Here are some of the finished products:

`/sponsored/i`


But let’s not get ahead of ourselves. Let’s start with the basics of Regex.

###How it works
At its most basic, regex is just writing expressions that try to match combinations of characters. Every string can be broken down into characters, so if why wouldn’t we be able to codify some kind of logic that allows us to build up every combination we see in the real world. That’s what the regex engine does.
Ruby recognizes regular expressions when you offset them by two backslashes, or with %r()

`/this is a regex expression/  
%r(and this is to!) `

Let’s look at some of the basic rules:

regex recognizes all of our basic characters, so if you were to just type in a word it would match:

![Basic Capture Image](../Images/Basic Capture.png)

There are also modifiers that you can add to the end of expressions, such as “i”, which allows you to recognize any case:

![Case Capture Image](../Images/Capture Case.png)

You can also capture ranges of characters, with the following syntax:

![All the Letters!](../Images/range of letters.png)

What you surround your capture group with matters as well. If you surround it with [brackets] it will match any character in the group. However, if you surround it with (parenthesis) it will only match the full word!

![Brackets](../Images/brackets.png) ![Parenthesis](../Images/Parenthesis.png)

Beyond basic characters, you can also match punctuation and white space:

![Periods](../Images/All the periods.png)![Whitespace](../Images/White Space.png)

These kinds of expressions are excellent for splitting up strings. The following piece of code does a decent job of splitting up strings by sentences. See if you can decipher the regex!

`string.split(/[.!?]/)`

###Quantifiers and Specials
Each of these individual expressions we’ve created are called “capture groups”. Unless otherwise specified - most expressions will capture when they’ve met their first match to their capture group.

If we look back at our expression to split strings, it works pretty well, but fails if we have two periods in a row:

`string = "This. should. split well... but it doesnt!!".split(/[.!?]/)  # => ["This", " should", " split", "", " but it doesnt"]`

The reason is because the capture group is matching each occurrence of the expression. We can fix this with the + quantifier. The + quantifier can be appended to a capture group to make it match 1 or more occurrences of the expression. This will chain all occurrences of the capture group that occur side by side into one capture. So we can fix our code like this:

`"This. should. split well... and it does!!!".split(/[.!?]+/)  # => ["This", " should", " split well", " and it does"]`

The plus quantifier can also be used to chain capture groups together:

![Chaining the Plus](../Images/plus chain.png)

There are also some special characters that regex uses to look at more complex concepts, such as words vs. not-words.
\w for words!
![Words](../Images/Words.png)

and \W for the not words
![Not Words](../Images/Not Words.png)

You can also do the same for digits with \d and \D

![Digits](../Images/digits.png)![Not Digits](../Images/not digits.png)

One more modifier that I think is really good to go over is the ?, which is a bit of a toughie. It is called the optional modifier. What it means is capture 0 or 1 of the preceding token. Here it is in action:

![? optional quantifier](../Images/optional quantifier.png)

However, ? can also mean lazy, which we’ll dive into below.

###Greedy Vs. Lazy
This is one of the tougher concepts to grasp in Regex, at least for me, so I’m going to do my best to explain it.

the * quantifier is called the “greedy” quantifier. It will match as many of the preceding character as possible when matching an expression. This is best displayed below:

Regular
![Regular](../Images/Regular.png)
VS. Greedy
![Greedy](../Images/Greedy.png)


Normally that expression would match anything with one character between two quotation marks. But because we placed a * quantifier, it matches as many characters as possible.


if ? is used on another quantifier, it can be made to make the group “lazy” or match as few characters as possible to make the capture effective. This is a bit of a mental leap, so just look at it in action below.

![Lazy](../Images/Lazy.png)



There are a ton of amazing things regex can do, way too much to go over in one blog post. But hopefully this gives you guys a good start, and if this sparked your interest, keep reading for a few great resources to learn more on your own!

[Regexer](http://regexr.com/) - how I did some of the images

[Regex One](http://regexone.com/) - a great tutorial that Kaylee found! Thanks Kaylee.
[RexEgg](http://www.rexegg.com/) - for those of you looking to master the whole thing and see how far down the rabbit whole you can go.
