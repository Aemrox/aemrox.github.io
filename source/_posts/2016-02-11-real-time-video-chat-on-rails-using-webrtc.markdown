---
layout: post
title: "Real-Time Video Chat on Rails Using WebRTC"
date: 2016-02-11 16:06:41 -0500
comments: true
categories: "Flatiron School"
---

There were many parts of my past projects that I genuinely enjoyed doing. I had moments that reaffirmed my entire choice to embark on this crazy career, and moments of unadulterated joy. Building video chat was not one of those moments. But I am happy I finally got it working.

The process was very much wading through the weeds, but now I wanted to chronicle the way I got this tech working with Rails.

A little background on WebRTC first (I don't want to go too deep into it, but you should know what you're working with). It's a free open-source framework start by Google for enabling real-time communication using nothing but native web browsers. The ability to support this kind of real-time communication with a piece of software that's available on essentially every computer is pretty amazing - but that doesn't necessarily mean it's easy to set up.

## The process from a Bird's Eye

Let me give you guys an overview of the technologies I used to get this up and running:

1. [WebRTC](https://webrtc.org/)
2. [Pusher](https://pusher.com/) - a 3rd party service for establishing real-time web sockets (along with the associated js library and ruby gem)
3. [Simplepeer.js](https://github.com/feross/simple-peer) - a javascript library for establishing secure connections between peers
4. [Hark.js](https://github.com/otalk/hark) - a javascript library for handling audio streams as speech

In broad strokes the process goes like this.

1. A user enters the link that hosts the video chat and is issued a GUID (or Global Unique Identifier)
2. The user's video is grabbed using webRTC's getUserMedia() function
3. The user establishes a connection to a pusher channel (or if they are the first party they establish the channel themselves)
4. The user then continually looks for any other peer's on the channel with whom they can connect
5. Another user enters the channel, following the same process from steps 2 and 3
6. Once the second member is detected, Simple Peer attempts to establish a connection between the two users
7. With a secure connection established, pusher handles the realtime transfer of video and audio streams between the two users and the respective feeds are assigned to video outlets in the html of the page handling the chat.
8. Use Hark to determine which channel is speaking, and ensure their video is being show.
9. BONUS - real-time messaging is enabled using basic DOM Manipulation and the open pusher channel between the two users.

I'll do my best to walk through each step of the process, exposing as much of my code as possible. Alternatively, you can just [take a look at the whole shebang](https://github.com/Aemrox/Edifi/blob/master/app/assets/javascripts/video_chat.js)

The number of guides I used is frankly hard to count and keep track of, but I'll try to include the lot of them in my resources section at the bottom. However, it's worth shouting out the [number 1 resource I've had in all of this: CarbonFive](http://blog.carbonfive.com/2014/10/16/webrtc-made-simple/). To be totally frank, a good portion of my code came from things this guy wrote - so this is a great source to look at if you have any holes left over from this tutorial.

##Step 1: Hosting the channel in a specific page
In order for all of this to work, you need an HTML page that has the appropriate sockets (here is [my example](https://github.com/Aemrox/Edifi/blob/master/app/views/chats/chat_page.html.erb)). In my code, the important sockets include:
1. a Div to eventually write remote videos
2. a ul for all videos
3. a preloaded video element for the local video (my own video)
4. most importantly, as a little workaround to send the pusher authentication info to the javascript, I place a little invisible div with some of the information about the pusher key. Before you scold me, I know this is insecure, but this is just getting a demo up and running.

Last part of this step, and the first part of the javascript that will handle the whole video chat is issuing the user a GUID. This is important because we need a way to uniquely identify every member of the chat when we look for new people joining the channel.

```javascript
var guid = (function() {
  function s4() {
    return Math.floor((1 + Math.random()) * 0x10000)
               .toString(16)
               .substring(1);
  }
  return function() {
    return s4() + s4() + '-' + s4() + '-' + s4() + '-' +
           s4() + '-' + s4() + s4() + s4();
  };
})();
```


Now I'm sure there are better more secure ways to do this, but I essentially just generate a really big number that is incredibly unlikely to de duplicated by the next person joining the chat. Since every chat channel is a unique pusher channel, the chances of this being an issue is infinitesimally small.

##Step 2: Grabbing the user's local video
This part is frankly fairly magical, and is part of what makes WebRTC so very cool. This is done in essentially one function:

```javascript
navigator.getUserMedia(mediaOptions, function(stream) {
  currentUser.stream = stream;
  var video = $('#localVideo')[0];
  video.src = window.URL.createObjectURL(stream);
  start();
}, function() {});
```

that getUserMedia() is where all the magic happens, you feed it some options defined earlier, and it returns a callback with a stream object which holds the real-time flow from the user's camera. All the permissions are handled by WebRTC, all of the proper wrapping of the video stream as well. That stream can be assigned to any html5 video element, as you see on line 4 - where I grab the local video element I designed in the page, and assign the video to it. At it's core, that's how easy it is to grab video through WebRTC - the real challenge comes in connection two people together.

Two notes before I move on:

Here are the options that need to be defined to determine how this stream is configured:

```javascript
var mediaOptions = {
  audio: true,
  video: {
    mandatory: {
      minWidth: 1280,
      minHeight: 720
    }
  }
};
```

Second, there is another line that needs to be written to make this work across all browsers, each of which alias webRTC in slightly different ways:

```javascript
navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
```

##Step 3: Establishing the Connection to Pusher
The first step here is to convert the user entering the channel into an object that pusher can work with. In my case, I pull the current user's name from the rails server with an ajax request. That user is then structured as such:

```javascript
currentUser = {
        name: data.user_name,
        id: guid(),
        stream: undefined //This gets defined when we pull the stream like we did in the previous section
      };
```

Now we need to create an object that will allow us to interact with pusher:

```javascript
var pusher = new Pusher($('#chat').data().apiKey, {
          authEndpoint: '/pusher/auth',
          auth: {
            params: currentUser
          }
        });
```
So there is a lot going on here that we need to unpack. In the first line we are instantiating a new pusher object (using the pusher.js library which should be included in your Rails' vendor javascript assets - I put it there manually).

That instantiation is using the Api Key which we hid in the html of the form. In a more complex (read secure) application, you would create a secure connection to the backend to retrieve that key, but let's just gloss over that for now.

In addition, that instantiation needs an authorization endpoint, which will exist as a route inside your rails app. Here is how that route is defined:

```ruby
match '/pusher/auth' => 'pusher#auth', via: :post
```

And here is what that route actually does:
```ruby
class PusherController < ApplicationController
  protect_from_forgery except: :auth

  def auth
    response = Pusher[params[:channel_name]].authenticate params[:socket_id],
      user_id: params[:id],
      user_info: { name: params[:name] }

    render json: response
  end
end
```


In addition, you need to make sure your rails app has your pusher key, secret, and app number as environment variables. For more on pusher setup with rails, take a look [here](https://github.com/pusher/pusher-http-ruby) at the pusher gem for rails. I took nearly all of the code for setup from there, so it's a good resource.

Now back to Javascript. We know have a pusher variable that represents our "pusher object". The next step is to find or create the appropriate channel, and connect to it:

```Javascript
var channelID = 'presence-chat-' + window.location.href.match(/lessons\/(\d+)\/?/)[1];
var channel = pusher.subscribe(channelID);
```

Here we take the url, pull the lesson number, and use that to name our nascent channel. This ensures that anyone in the same link will be on the same pusher channel. We then assign that channel to a variable, using the pusher function subscribe(). This will find the channel if it exists, or open it for the first time. For more on how this works, check out the [Pusher documentation](https://pusher.com/docs).

Now we have our channel open for at least one user, we will be using this channel variable as our main point of entry from here on out, so get used to it.

##Step 4: Looking for others on the channel
This step is fairly simple, we are going to write a short little loop, that will look for other users as long as the channel is open:

```javascript
var peers = {};
function lookForPeers() {
  for (var userId in channel.members.members) {
    if (userId != currentUser.id) {
      var member = channel.members.members[userId];

      peers[userId] = initiateConnection(userId, member.name);
    }
}
```

This is a for loop that loops through the guids of all of the members of the channel. If it finds a guid that is not equal to the guid of the current user, it initiates a connection (a piece of code we will see later).

This is why it was important to establish a guid when we created our user. Now, we are going to have to define when this function is called. The way we do that is by binding it as a callback to be called when the channel has been successfully subscribed to, as such:

```javascript
channel.bind('pusher:subscription_succeeded', lookForPeers);
```

The concept of binding actions to client events (such as a successful subscription to a channel) is part of what makes pusher so powerful and robus - if you are interested in learning more, take a look at [this piece of documentation](https://pusher.com/docs/client_api_guide/client_events).

##Step 5: A Wild User Appears!
The trick to how this code really works is that it is embedded in the web page. So whenever a new user enters the page, they begin the same process. So in our imaginary case, another person will enter the channel, and go through the steps 2 and 3 and end up subscribing to the same pusher channel.

Once this is done, our callback to look for other peers will be called, initiating the process of connecting our first and second user.

##Step 6: The Handshake and then some
Now that we have two users occupying the same pusher channel, we need to establish a connection. Establishing any secure connection can be an arduous process, so finding a library that does it for us makes this a lot easier. The library I chose was a fairly easy one to use called [SimplePeer](https://github.com/feross/simple-peer).

Like with most libraries, easiest way to make this work is to pop it in the javascript folder in your vendor directory within the rails app. From there, use is pretty simple.

As we saw earlier, once we find another peer, we are going to call a function called initiateConnection. Here is the function in it's entirety:

```javascript
function initiateConnection(peerUserId, peerUserName) {
  return setupPeer(peerUserId, peerUserName, true);
}
```

The reason for having this spacing between the initiation of the handshake and initiate connection is to be able to determine who is the owner of the connection, or who is making it. This initiateConnection function is being called by the person looking for peers, so whoever is setting up this connection is the initiator.

setupPeer() will need to be called by both ends of the connection, so let's take a look at setupPeer(). Fair warning, it's a lengthy bit of code:

```javascript
function setupPeer(peerUserId, peerUserName, initiator) {
  var peer = new SimplePeer({ initiator: initiator, stream: currentUser.stream, trickle: false });

  peer.on('signal', function (data) {
    channel.trigger('client-signal-' + peerUserId, {
      userId: currentUser.id, userName: currentUser.name, data: data
    });
  });

  peer.on('stream', function(stream) { gotRemoteVideo(peerUserId, peerUserName, stream) });
  peer.on('close', function() { close(peerUserId, peerUserName) });
  $(window).on('beforeunload', function() { close(peerUserId, peerUserName) });

  peer.on('message', function (data) {
    if (data == '__SPEAKING__') {
      $('#remoteVideos video').hide();
      $("#remoteVideos video[data-user-id='" + peerUserId + "']").show();
    } else {
      appendMessage(peerUserName, data);
    }
  });

  return peer;
}
```
The first thing that's happening is that we are using simple peer to create a peer object. This object is aimed at mirroring the other user, almost like the currentUser variable we set up in the beginning. What happens next is that we assign this peer many messages it can respond to, along with what it should respond with.

The first message is the signal test, this triggers a ping and response between the currentUser and their peer. This is used to establish the connection in the other direction (for the non-initiator). The event that this behaviour triggers is bound later in the following code:

```javascript
channel.bind('client-signal-' + currentUser.id, function(signal) {
  var peer = peers[signal.userId];

  if (peer === undefined) {
    peer = setupPeer(signal.userId, signal.userName, false);
  }

  peer.on('ready', function() {
    appendMessage(signal.userName, '<em>Connected</em>');
  });
  peer.signal(signal.data);
});
```

The next message is a stream, it is the basic request for the video stream, which triggers the gotRemoteVideo function which we will take a look at in a second.

The next message is close, which severs the connection. If you take a look at the next line of code, it ensures that if the window ever closes the connection is automatically severed.

The rest of the code revolves around a couple of bonus functions, the first is live messaging between the two. The other is making sure that whoever is speaking has their video featured on the remote video feed. We'll revisit these later in our bonus section.

##Step 7: Gettin' Dat Video
So as we've seen both when we create our user and establish the connection with our peer, we are always setting a stream object that we've taken from webRTC. That stream object is about to be put to use to complete our video chat. So once the stream has been received, we need to render it into usable video. Thankfully, that is fairly easy to do:

```javascript
function gotRemoteVideo(userId, userName, stream) {
  var video = $("<video autoplay data-user-id='" + userId + "'/>");
  video[0].src = window.URL.createObjectURL(stream);
  $('#remoteVideos').append(video);

  var preview = $("<li data-user-id='" + userId + "'>");
  preview.append("<video autoplay/>");
  preview.append("<div class='name'>" + userName + "</div></li>");
  preview.find('video')[0].src = window.URL.createObjectURL(stream);

  $('#allVideos').append(preview);
}
```

So here we are creating an html video object, setting it's feed to the stream object we received, and appending it to the big screen (#remoteVideos). In addition, we are creating a smaller preview object to place at the bottom of the screen (much like they do on Google Hangouts).

And there you have it! Two way video! Obviously there are lots of places this can go wrong, especially in the connection pusher. Luckily, pusher provides a very handy interface to handle all these debugging problems, and checking that status of the connections you've made. Just checkout the debug console of your [pusher dashboard](https://dashboard.pusher.com/).

##Steps 8 & 9, HAVEN'T YOU HAD ENOUGH ALREADY?!
So there are a couple other tricks that this code does to replicate the functionality of something like google hangouts.

The first is using Hark.js, a library that can identify speech, to send a signal back and forth that indicates when someone is speaking.

```javascript
var speech = hark(currentUser.stream);

speech.on('speaking', function() {
  for (var userId in peers) {
    var peer = peers[userId];
    peer.send('__SPEAKING__');
  }
});
```

This signal (which we've actually already seen!), will trigger a hiding and showing of the correct remote video feed.

The messages is a bit more complicated, I have not gotten it to work in both directions 100%. But you've already seen a large chunk of the code. Here is the last bit:

```javascript
$('#send-message').submit(function(e) {
  e.preventDefault();
  var $input = $(this).find('input'),
      message = $input.val();

  $input.val('');

  for (var userId in peers) {
    var peer = peers[userId];
    peer.send(message);
  }
  appendMessage(currentUser.name, message);
});
```
There is a message bar that we've defined on our html page. When a message is submitted, it is sent out to every peer. As we've seen in the listening functions we've set on a peer, when that message is received, it is appended to the message box at the bottom of the screen.

The truth of the matter is, all of these bells and whistles are fairly simple once you have the connection established. Pushed is an incredibly powerful piece of technology, that I would heartily recommend exposing yourself to!

##TLDR
![VIDEOOOO](https://media.giphy.com/media/iNwXHhkqlfOla/giphy.gif)

##Resources:
- [Pusher's Tutorial on WebRTC](https://pusher.com/tutorials/webrtc_chat)
- [CarbonFive's WebRTC Setup](http://blog.carbonfive.com/2014/10/16/webrtc-made-simple/)
- [Simple Peer](https://github.com/feross/simple-peer)
- [WebRTC Basics from HTML5Rocks](http://www.html5rocks.com/en/tutorials/webrtc/basics/)
- [Pusher Documentation](https://pusher.com/docs)
