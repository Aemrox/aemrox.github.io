---
layout: post
title: "The Wonderful World of Regex"
date: 2015-12-08 13:52:56 -0500
comments: true
categories: "Flatiron School"
---

While working on my last project, our main goal was solving one very difficult problem.

While ingesting around 1,000 articles a day, we needed to add some semblance of location data to each one of those articles. When we first launched, this was done by me. Manually. All. Goddamn. Day.

I would manually scan the article for some kind of address, or area (like 5th ave between 42nd and 43rd). We noticed that I was following very similar patterns - addresses are generally clean patterns, and areas often use the same prepositions over and over again. My partner figured that we could begin parsing these texts automatically with a series of rules.

Since I was not any kind of developer at the time, I was less involved with the coding of the solution, but one way I was able to chip in was in writing the logic of finding these phrases through something called Regex.

Regex stands for Regular Expressions, and is a very powerful tool used for identifying, matching and capturing certain patterns in text and strings.

We would write regular expressions to begin automatically filtering and flagging the articles we were pulling in for items that didn’t fit what we were doing, like sponsored advertising:

`/sponsored/i`

But more importantly, we wrote a series of expressions that would look for the prepositions connecting areas, or the patterns of addresses. Here are some of the finished products:

`\b([0-9]+ (West|W|East|E|North|N|South|S) [0-9]+(st|nd|rd|th))\b`

`((East|West)? ?([0-9]+|[A-Z][a-z]+|first|second|third|[a-z]+th)(st|th|nd|rd)? ?(Road|road|rd|Rd|Street|street|St|st|Avenue|avenue|Ave|ave|Place|place|Pl|pl|Parkway|parkway|pk|Pkwy|pkwy|Drive|drive|Dr|dr|Boulevard|boulevard|Blvd|blvd|Way|way|Lane|lane|Ln|ln|Court|court|Plaza|plaza|Square|Terrace|terrace)) +(near|and|at) +(Avenue [A-Z])\b`



But let’s not get ahead of ourselves. Let’s start with the basics of Regex.


###How it works
At its most basic, regex is just writing expressions that try to match combinations of characters. Every string can be broken down into characters, so we should be able to codify some kind of logic that allows us to build up every combination we see in the real world. That’s what the regex engine does.

Ruby recognizes regular expressions when you offset them by two backslashes, or with %r()

`/this is a regex expression/  
%r(and this is to!) `

and these expressions can be entered in String.#methods such as #Scan or #Split

##Basic Rules

Regex recognizes all of our basic characters, so if you were to just type in a word it would match:

![Basic Match Image](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Basic%20Match.png?raw=true)

There are also modifiers that you can add to the end of expressions, such as “i”, which allows you to recognize any case:

![Case Capture Image]https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Case%20Match.png?raw=true

You can also capture ranges of characters. Capture any lower case character with the following syntax:

![match all lower case letters](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Range%20Match.png?raw=true)

the little g modifier at the ends means it is global, and will match every instance across the string. If we were to remove it, it would only match the first instance.

Beyond basic characters, you can also match punctuation and white space by using escape characters:

![periods](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Periods.png?raw=true)
![whitesspace](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/White%20Space.png?raw=true)

The reason for needing the escape character is that there are special characters which denote larger sets, such as the “.” which matches any character

![Dot](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Dot.png?raw=true)

There are also some special characters that regex uses to look at more complex concepts, such as words characters
\w for words!
![words characters](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/word%20characters.png?raw=true)

You can also do the same for digits with \d

![digits](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/digits.png?raw=true)


##Quantifiers
Unless otherwise specified - most capture groups will capture each instance of a match as an individual capture. However, we can use several quantifiers to modify how capture groups behave and give them more (or less) flexibility

If we look back at our expression to split strings, it works pretty well, but fails if we have two periods in a row:

`string = "This. should. split well... but it doesnt!!".split(/[.!?]/)  # => ["This", " should", " split", "", " but it doesnt"]`

This doesn’t do exactly what we want because the capture group is matching each occurrence of the expression. We can fix this with the + quantifier. The + quantifier can be appended to a capture group to make it match 1 or more occurrences of the expression. This will chain all occurrences of the capture group that occur side by side into one capture. So we can fix our code like this:

`"This. should. split well... and it does!!!".split(/[.!?]+/)  # => ["This", " should", " split well", " and it does"]`

Another excellent modifier is the * or splat. When placed inside a capture group, it takes the preceeding token, and tells regex to match as many of these as possible.

![Splat](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Splat.png?raw=true)

One more modifier that I think is really good to go over is the ?. It is called the optional modifier. What it means is capture 0 or 1 of the preceding token. Here it is in action:

![optional modifier](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Optional%20Modifier.png?raw=true)

However, ? can also mean lazy, which we’ll dive into below.

##Greedy Vs. Lazy
This is one of the tougher concepts to grasp in Regex, at least for me, so I’m going to do my best to explain it.

the * quantifier is called the “greedy” quantifier. It will take as many characters as necessary to create a capture group. To understand this, look at the following string.

“You are becoming quite skilled, young grasshopper,” said Steven with glowing admiration in his eyes. “quite skilled indeed.”

If I wanted a regex expression to match “anything between the quotes” - it actually has several groups it could capture. It could do both traditional quotes (as we see them), it could take the inverse, which is the area between the 2nd and 3rd quotation mark, or it could take the whole sentence.

So a * tells regex, take as many characters as possible, or, be greedy.

This is best displayed below:

![greedy](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Greedy.png?raw=true)

This doesn’t get us all the text in quotations as we see them. B ecause we placed a * quantifier, it matches as many characters as possible.

if ? is used on another quantifier, it can be made to make the group “lazy” or match as few characters as possible to make the capture effective. This is a bit of a mental leap, so just look at it in action below.

![lazy](https://github.com/Aemrox/aemrox.github.io/blob/source/source/images/Lazy.png?raw=true)

There are a ton of amazing things regex can do, way too much to go over in one blog post. But hopefully this gives you guys a good start, and if this sparked your interest, keep reading for a few great resources to learn more on your own!

http://regexr.com/ - how I did some of the images

http://regexone.com/ - a great tutorial that Kaylee found! Thanks Kaylee.
http://www.rexegg.com/ - for those of you looking to master the whole thing and see how far down the rabbit whole you can go.
