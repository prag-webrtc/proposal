Stolley: Sample chapter

# Chapter Two: Establishing a Peer-to-Peer Connection

WebRTC’s possibilities are a lot easier to imagine once you’ve wired up a connection and can see it in action across a couple of open browser windows on your desktop. The rest of the book looks more closely at all the different components of the peer-to-peer web applications and the incredible things you can build. Other chapters will also include a lot of invitations for you to experiment. But the goal in this chapter is to work systematically and get some foundational code working as quickly as possible.

You’ll be building a basic video-chat app that allows two connected peers to stream video directly to each other. (To prevent skull-shattering feedback while you test things out, you’ll start off with the audio disabled.) In the process, you will do meaningful work with the core pieces of a peer-to-peer app. We’ll start off by building a basic interface in semantic HTML and leading-edge CSS before turning attention to the media capture API, media streams, and a signaling server. The chapter then walks you through building a real-world implementation of the “perfect negotiation” pattern found in the WebRTC specification for setting up a connection with the `RTCPeerConnection` interface.

For development purposes, you’ll want to use the latest version of either Chrome (version 80 or higher) or Firefox (version 75 or higher). I personally prefer Firefox Developer Edition, but the choice is yours. The only modern browsers to avoid for development, at least at first, are Safari and versions of Edge older than 80. In a later chapter, you will add just a bit more code to make the perfect negotiation pattern backward compatible with a few less-than-perfect modern browsers, like Safari, that are slightly behind the curve.

## Getting and Using the Supporting Code
You can find the starter code for this chapter on GitHub at https://github.com/prag-webrtc/ch-2-starter. If you like, you can download it as a ZIP file, unzip it, and put it in your folder of choice. Make sure to save it someplace you can reach easily from the command line.

If you prefer to work with Git, you can clone the repository to any location you like by firing up your terminal and running this command:

```sh
$ git clone git@github.com/prag-webrtc/ch-2-starter.git
```

If you want to have a completed version of the code either to follow along with or to check your own work against, get yourself a copy of the repository at https://github.com/prag-webrtc/ch-2-complete. There is also a live version of this app available at https://rtc.stolley.dev/.

Once you have a copy of the repository, take a look around and find the directory or folder called  `public/`. It contains the HTML, CSS, and JavaScript files for the chapter’s app. The HTML you’ll be writing belongs in `index.html`, right inside of `public/`. The CSS and JS files are in the `css/` and `js/` subdirectories inside of `public/`. You’re welcome to use those starter files as-is, or empty them out and start your own.

If you already set up Node.js in Chapter One, you can quickly install some dependencies and then fire up a server to test your files in your development browser as you work. On the command line, change directories to the one that contains the `server.js` file and run these commands:

```sh
$ npm install
$ npm run dev-start
```

A bunch of output will fly past in your terminal while the `npm install` command handles all of the project dependencies.

The next command, `npm run dev-start`, will output a few lines of diagnostics in your terminal, ending with some lines that look like these:

```sh
  socket-express:server  Available on:
  socket-express:server   * https://127.0.0.1:3000
  socket-express:server   * https://192.168.1.6:3000
  socket-express:server  Hold CTRL + C to stop the server
  socket-express:server  +0ms
```

With that information in hand, you can open your browser to `https://localhost:3000/`. Of course `localhost` is just a shortcut for `127.0.0.1`, which you can also use if you prefer typing numbers and dots: `https://127.0.0.1:3000/`. Don’t worry for now about the second IP address you see. It will almost certainly be different from `192.168.1.6`, but eventually you will be able to use whatever that second address is to test your app with other devices connected to your local network.

## Structuring and Styling a Basic Peer-to-Peer Interface
Let’s begin with a basic process for building something new for the web: establishing a workable UI concept, structuring it in semantic HTML, and styling the HTML with just enough CSS to keep it easy on the eyes.

### Peer-to-Peer UI Patterns
To start things off, consider the kinds of interfaces that are common in peer-to-peer apps you have used. Their interfaces are generally modeled on one of two patterns: a caller pattern, or a joiner pattern. A caller pattern is asymmetrical: Certain peer-to-peer apps, like FaceTime or Skype, and even the telephone, depend on one peer calling and the other answering. The FaceTime interface, for example, looks the same for two peers in- and outside of a call. But while a call is being placed, the interface is different for both the caller and answerer. The interfaces return to a symmetrical state once the person being called makes a decision: either to answer, causing both interfaces show the call, or to decline (or simply ignore), in which case both interfaces return to how they looked before the call was placed.

Other peer-to-peer apps, like Google Meet or Zoom, are symmetrically patterned: instead of setting up a call between a caller and answerer, symmetrically patterned apps treat a call as something that already exists for users to join whenever they’re ready. The interface depends only on what any one user is doing: preparing to join a call, participating in a call after joining, or leaving the call.

WebRTC-backed interfaces can be constructed using either pattern, caller or joiner. But the joiner pattern simplifies the interface: there’s no need to create a screen to handle an incoming call, or wire up a bunch of buttons to answer or decline. Instead, one button is all that’s needed—Join Call—and it’s presented to each peer. In the joiner pattern, we don’t even have to think in terms of someone calling someone else. From the perspective of the peers on the call, no one cares who joined first. As the designers of the app, we don’t have to care, either.

### Adding a Header and Button
With a joiner pattern in mind, you can start building the HTML and CSS to structure and style the page. No need to worry about special HTML, CSS, or JavaScript for a caller or an answerer: everyone on the call is just a joiner, so everyone gets the same code.

Start things off in the `index.html` file with a simple header that has a first-level heading along with a button element for joining the call. The button is going to enable users to join and leave the call, but you can set it up initially as a join button. When the page first loads, the call will need to be joined:

```html
  <header id="header">
    <h1>Welcome</h1>
    <button type="button"
      id="call-button"
      class="join">Join Call</button>
  </header>
```

The `type="button"` attribute might look redundant, but it’s good practice to include it for any button element without a browser-provided behavior: `<button>` defaults to `type="submit"`, for submitting form data, in the absence of an explicit `type` attribute. But this button is not submitting a form.

The unique ID `call-button` is deliberately generic. It will save a lot of work we would otherwise have to do in JavaScript to swap in separate join and leave buttons. The class `join`  will work as a nice styling hook in CSS and, later, for JavaScript to determine the state of the button. The button’s “Join Call” text should be an unambiguous cue to users. In a bit, a little JavaScript will change the button’s class to `leave` and its text to “Leave Call” once the “Join Call” button has been clicked.

### Video Elements: Self and Peer
With a basic heading and button in place, the only other HTML this basic app needs is for handling video: one element for a user’s own video stream, which we’ll refer to as *self*, and one element for the remote user’s video stream, which we’ll refer to as *peer*.

One of the many fun aspects of working with peer-to-peer streaming media is that you get to write some things in your markup that are usually big no-nos. One such thing is setting up multiple video elements to play simultaneously (one for self, one for the peer), with some attributes that might surprise you if you’re familiar with the `<video>` element introduced in HTML5. If you’re not familiar, that’s okay, too. We’ll walk through them.

This is how to set up the video element for the self video, which for convenience takes an ID of `self`:

```html
<video id="self"
  autoplay
  muted
  playsinline
  poster="/images/placeholder.png">
</video>
```

That `<video>` element sets a number of important attributes. Let’s briefly explore each one’s purpose in the context of live-streaming video:

#### The `autoplay` attribute
Ordinarily the `autoplay` attribute is frowned upon. Users generally expect control over the playback of video and especially the accompanying audio. Browser makers have helped to enforce user control by disregarding the `autoplay` attribute until a user has interacted with the page in some way first. But for streaming video, `autoplay` is strictly necessary: without it, the browser will only the show very first frame of a streaming video. And because users will have to click the “Join Call” button you built above, they will have interacted with the page before any media begins to stream—all but guaranteeing that the browser will respect the `autoplay` attribute.

#### The `muted` attribute
Initially, for same-machine testing purposes, we’ll exclude audio from the streaming tracks entirely. But to ensure that future audio-capable streams don’t cause hellacious feedback, the self video takes the `muted` attribute. That’s different from muting your mic so no one else can hear you—a topic we will get to in a later chapter. All this attribute does is disable audio on the self video.

#### The `playsinline` attribute
The final Boolean attribute to include is `playsinline`. This attribute instructs mobile devices in particular not to launch a full-screen presentation of the video once the stream starts, which is the default behavior in Safari on iOS and other mobile browsers. While full-screen video might sound desirable, it will obscure anything else on the page, including any user-interface components and the self video. With `playsinline` set, the peer video will play wherever on the page you place it with CSS.

#### The `poster` attribute
And although not strictly necessary, a placeholder image can be set on the `poster` element. It will display until the video stream starts, which is a basic way to prepare users to expect to see video streams on the page. It’s helpful for testing purposes too: without a poster or some explicit dimensions and colors set in CSS, video streams would appear out of apparent nothingness—or not appear at all, if there’s something wrong with the peer connection.

### The Peer Video
The markup for the peer video is almost identical to the self video, except that it takes an ID of `peer` and omits the `muted` attribute:

```html
<video id="peer"
  autoplay
  playsinline
  poster="/images/placeholder.png">
</video>
```

You might have noticed that there is neither a `src` attribute nor any inner `<source>` elements for either video. Their omission is intentional. There’s no file involved in streaming video. Later in this chapter you’ll use JavaScript to set up the streaming source for each video element, using the `srcObject` property.

Putting it all together, your HTML file should look something like this:

```html
  <header id="header">
    <h1>Welcome</h1>
    <button class="join" type="button"
      id="call-button">Join Call</button>
  </header>
  <video id="self"
    autoplay
    muted
    playsinline
    poster="/images/placeholder.png">
  </video>
  <video id="peer"
    autoplay
    playsinline
    poster="/images/placeholder.png">
  </video>
```

That’s it: a heading, a button, and two video elements. Let’s get them styled up in CSS.

### Styling the Page and Header Elements
With the HTML in place, bare bones as it is, you can turn your attention to setting up some CSS to present a workable interface. I always begin my work with Eric Meyer’s Reset CSS. I won’t replicate that here, but you will see a minified version of it in the starter `screen.css` file for this project.

I prefer to begin by setting up a page’s basic typographic properties, usually on the `html` selector:

```css
html {
  font-family: "Lucida Grande", Arial, sans-serif;
  font-size: 18px;
  font-weight: bold;
  line-height: 22px;
  padding: 44px;
}
```

That text setting might strike you as a bit large and bulky, especially because these are also meant to be mobile-first styles. That is deliberate: think about your face’s position relative to your screen when you’re on a video call. Most of us move back from the screen so that the camera can capture at least our entire heads. And if your head is a prize-winning pumpkin like mine, you might really have to move back. With users’ eyes further back from the screen, it’s a more accessible choice to err on the side of an oversize, easily visible UI.

The `<button>` element, the only interactive UI on the page, is generally easier to style than other form elements. You can set up a basic look for it on the `button` element selector, opening with a bunch of font styles to inherit the look of all text on the page:

```css
button {
  font-family: inherit;
  font-size: inherit;
  font-weight: inherit;
  line-height: inherit;
  /* Box Styles */
  display: block;
  border: 0;
  border-radius: 3px;
  margin: 11px 11px 22px 0;
  padding: 11px;
}
```

For maximum control over the button’s styling, it’s worth setting it to display as a block. I’ve removed the default border that browsers draw, and given the button a small border radius. A little bit of margin and padding—derived from the page’s 22px `line-height` value—provide some room around the text and some margins to offset the element. Nothing fancy. (And don’t worry—if your familiarity with responsive design has you feeling a gnawing guilt for using pixel units, you’ll get to explore and implement responsive units in the context of peer-to-peer interfaces in a later chapter.)

The button’s HTML currently only has the `join` class, but you can still set up some very basic colors on both the join and leave classes for the button:

```css
.join {
  background-color: green;
  color: white;
}
.leave {
  background-color: #666;
  color: white;
}
```

Finally, although it’s overkill to use ID selectors in CSS like this, for the sake of simplicity you can specify a pointer style and a width on the `#call-button` selector:

```css
#call-button {
  cursor: pointer;
  width: 143px; /* 6.5 grid lines */
}
```

The 143px width value is tailored to comfortably fit the width of “Join Call” and “Leave Call,” while also being derived from the page’s 22px line-height. All the width does is ensure the button won’t jump widths when clicked. That doesn’t matter much when the button on its own line, but as a small responsive touch, let’s add a media query to display the contents of the `<header id="header">` as flex items, all on the same line:

```css
@media screen and (min-width: 500px) {
  #header {
    display: flex;
    align-items: baseline;
    flex-direction: row-reverse;
    justify-content: flex-end;
  }
  #header > * {
    flex: 0 0 auto;
  }
}
```

If you’re not familiar with flexbox and its properties, what’s happening here is the page header is set to display as a flexbox with `display: flex`. Aligning the flex items—the first-level heading and the button—to the baseline keeps the text of each on the same invisible line.

Then, to move the join button so that it sits to the left of the heading, `flex-direction: row-reverse` flips the order of the header’s items, while `justify-content: flex-end`, on a row-revered flexbox, will keep the flex items aligned to the *left* of the flexbox.

The child selector `#header > *` sets the behavior of the flex items (an `<h1>` and `<button>`, in this case). The shorthand `flex` sets the grow and shrink values to zero, meaning that neither element will grow or shrink from its `auto` width. For the first-level heading, that will be the width of its text. For the button, that will be the 143px value of the `width` property set on the `#call-button` selector.

The result is that on viewports 500 pixels and wider, the button and heading sit on the same line, which will reduce how far down the viewport they push the video elements. And speaking of the video elements, let’s style them next.

[FIGURE PLACEHOLDER]

### Styling the Video Elements
As embedded content, the `<video>` element displays inline by default, just like the `<img>` tag does. Ethan Marcotte made famous the pairing of `display: block` and `max-width: 100%` for responsive images, and we can do the same for videos. Setting the `max-width` property ensures that the video element will be no larger than either its parent element’s width or, in the case of this small app’s HTML, the viewport:

```css
video {
  background-color: #EEE;
  display: block;
  max-width: 100%;
}
```

I’ve set the background color on video elements to a very light shade of gray, which will show the exact boundaries of the video elements and play nicely with the transparent smiley-face PNG set on the video element’s `poster` attribute.

The peer video will display as-is, but a few adjustments to the self video will make it obvious which video is which. That’s helpful if you have just a single camera on your computer. The same streaming video image will appear in both videos when you’re testing:

```css
#self {
  width: 50%;
  margin-bottom: 11px;
}
```

This reduces the self video’s width to 50%, and puts a half `line-height` of space between it and the peer video that will sit below it.

With all of that CSS in place, you can reload the page in your browser to see something like this:

[FIGURE PLACEHOLDER]

## Adding Functionality to the Call Button in JavaScript
That’s all the structure and styling your app needs. Now it’s time to cozy up to JavaScript. Let’s start by reviewing a data structure in JavaScript that will be very useful to keeping yourself organized: object literals.

In my own work, I’m a big fan of object literals for grouping related items. Mostly because I hate naming a lot of variables, and keeping track of them all. You might have seen object literals in JavaScript before. For example, here’s a simple object literal for holding details of my dog, Hank:

```javascript
var dog = {
  name: 'Hank',
  age: 7
}
```

Object literals are useful for many reasons, but they really do cut down on the work and complexity of managing a large number of freestanding variables. Instead of `dogAge` and `dogName` variables, object literals let us reference properties on a single variable: `dog.age` and `dog.name`.

If there’s a need to add additional properties to an object, or change existing ones, all we have to do is assign them:

```javascript
dog.bestFriend = 'Fanny';
dog.age = 8; // Happy Birthday, Hank!
```

Let’s look at a more practical example. Creating interactive DOM elements is a routine part of any JavaScript file, so I habitually group those all into an object literal called `ui`, which—shocker—stands for *user interface*. To keep my JavaScript files organized in development, I like to open them by declaring all of my global variables, starting in `main.js` with `ui` assigned to an empty object literal:

```javascript
// main.js
var ui = {};
```

You can then use the `querySelector()` method on the `document` object to select the call button from the HTML. If you’ve not used `querySelector` before, know that it takes the same syntax as you’d write in CSS to reference an element in the HTML:

```javascript
// main.js
var ui = {};
ui.callButton = document.querySelector('#call-button');
```

With `ui.callButton` holding onto the reference for the call button, you can then call the `addEventListener()` method to respond to click events on the button. Start simple and just report in the console that the button has been clicked:

```javascript
// main.js
ui.callButton.addEventListener('click', function() {
  console.log('Call button clicked!');
});
```

Reload your page in the browser and open the JavaScript console. You should see “Call button clicked!” appear in the console, like this:

[FIGURE PLACEHOLDER]

### Named Functions as Callbacks
While the `querySelector` method took a single argument—the CSS selector `#call-button`—the `addEventListener` method takes two arguments. The first argument is the name of the event, `'click'`, and the second argument is an *anonymous* callback function, also known as a *listener*. The function is *anonymous* because it doesn’t have a name: the `function` keyword just defines the function in place. I like callback functions to have names and their own place to live in a JavaScript file, so the first little piece of refactoring here is to create a named function and pass its name in as the second argument to `addEventListener()`:

```javascript
// main.js
ui.callButton.addEventListener('click', handleUiCallButton);

/*
  DOM-event Callback Functions
*/
function handleUiCallButton() {
  console.log('Call button clicked! Named callback function active!');
}
```

Reload your page in the browser and click the button once again. The console should show the longer message announcing that the named callback function is active.

If you saw the message in the console *before* you clicked, it’s possible that you might have called the function by mistake:

```javascript
// main.js
ui.callButton.addEventListener('click', handleUiCallButton()); // Oops!
```

If you included the opening and closing parentheses, `()`, on `handleUiCallButton`, what gets passed into `addEventListener` is not the callback function itself, but the *result* of the callback function after it runs prematurely. If that happened, make sure that you’re passing in the name of the function only: `handleUiCallButton`, without parentheses. Then refresh the page in your browser and try again.

With the named function passed in as a reference, our code looks a little cleaner: we can simply see the event, `click`, and the descriptive name of the function to be called in response to the event:

```javascript
// main.js
ui.callButton.addEventListener('click', handleUiCallButton);
```

The nitty-gritty details of `handleUiCallButton` are in their own spot toward the bottom of the JavaScript file. That organizational habit will be even more important once the file includes more of the WebRTC code, which is almost entirely event-driven.

### Working with Data Returned to Callback Functions
The `addEventListener` method, like many methods that work with callback functions, passes along chunk of data to the callback when its event fires. With  `addEventListener` , it’s basically convention to handle that data using either `event` or simply `e` as an argument in the callback function:

```javascript
// main.js

function handleUiCallButton(event) {
  console.log('Button with ID', event.target.id, 'clicked!');
}
```

Refresh and click the button again, and watch for “Button with ID call-button clicked!” to appear in your console.

The `target` property on `event` is a DOM object, much like we’d get from using `document.querySelector()`, which means we can do exactly what we need with the button:

1. Change its class from `join` to `leave`, and vice-versa
2. Change its text from “Join Call” to “Leave Call,” and vice-versa

Let’s set that up. To make the code more readable, hang on to the `event.target` value in a local variable called `callButton`:

```javascript
// main.js

function handleUiCallButton(event) {
  var callButton = event.target;
  if (callButton.className == 'join') {
    console.log('Joining the call...');
    callButton.className = 'leave';
    callButton.innerText = 'Leave Call';
    // TODO: Add all the things to actually join a call...
  } else {
    console.log('Leaving the call...');
    callButton.className = 'join';
    callButton.innerText = 'Join Call';
    // TODO: Add all the things to actually leave a call...
  }
}
```

Refresh your page yet again. Now you can click the button again and again. It’s text will change and, thanks to CSS that you wrote earlier, so will its color.

[FIGURE PLACEHOLDER]

The console will also log “Joining the call…” or “Leaving the call…,” depending on what state the button is in.

Excellent. All of the foundational HTML, CSS, and JavaScript for the video-chat app interface is now in place. What’s left is the JavaScript to create a peer-to-peer connection. But before we write any more, let’s take a code break and look a little bit at WebRTC as implemented in the browser.

## WebRTC is a Front-End Technology
With an imposing name like *Web Real-Time Communication Between Browsers*, the WebRTC specification sure sounds like a server-based technology. As web developers, we’ve simply never seen anything *between* browsers that doesn’t require a server. A server has always been what a browser talks to: basic requests for static HTML pages, processing data submitted with a form, or asynchronous requests with the Fetch API. The traditional web has always been a 100% server-backed technology.

The web relies on a client-server architecture, where the *client* part is usually synonymous with *browser*. (Web-standards documents refer to *user agents*—which are clients or browsers, too.) Client-server architecture means that even something as basic as a text-based chat application requires relaying each chat message from a browser, through a server, and onto the receiving browser. Web APIs like WebSockets have enabled persistent connections and push-like behavior, as we reviewed in Chapter One, so that chat messages appear to come across instantaneously from one user to another. But even with WebSockets, the basic architecture of the web, including the server as the central hub, remains unchanged:

[FIGURE PLACEHOLDER]

As a genuine peer-to-peer technology, the architecture of WebRTC just about eliminates the need for a server. The suite of APIs that comprises WebRTC are all implemented natively in the browser. (There are server, OS, and even smart-device implementations of WebRTC, too, but they represent very different use cases, and are beyond the scope of this book.) Once a connection is established, WebRTC in the browser enables any two peers to directly exchange data, including streaming media, without that data ever touching a web server. This is the browser-to-browser web.

### Peer-to-Peer Connections Must Be Negotiated
Of course, there’s one little hitch: WebRTC is peer-to-peer *once a connection is established.* RTC peer connections take a little bit of work to establish, of course, and usually with the aid of a server.

As web developers, we’re used to the request-response connection pattern of HTTP: if we have a domain name or even just an IP address, we just enter it in the address bar of the browser and boom—connection established and resource or error message delivered in response to the request. End of story. You’ve been doing that already with the little local web server for previewing your work on the video-chat app’s interface. Subsequent requests to the same website work the same way as the first request, but typically without a persistent connection between browser and server unless you’re dealing with something fancy like WebSockets.

Establishing a connection with WebRTC is a bit more involved than request-response. The two connecting peers have to negotiate the terms of the connection before it can be established. Instead of request-response, peer connections are negotiated by an offer-answer pattern over a signaling channel.

The offer-answer structure of establishing an RTC peer connection is metaphorically no different from a conversation between two friends who want to meet for coffee. Unless they share a remarkably strong psychic connection, the two friends won’t just show up the same day and time at the exact same coffee shop. They must first negotiate the details of the coffee date, using a some kind of a signaling channel: email, phone, text, or something conventional like that. Even a remarkably strong psychic connection represents a signaling channel, when it comes right down to it.

But let’s keep it conventional and familiar: One friend initiates the coffee date by sending a text, making a phone call, or whatever. That friend provides proposed details on a plan to meet up, and the other friend responds. The friends go back and forth, offering and answering, until they reach an agreement to meet at a specific time and place. Regardless of the signaling channel used (text, phone, email), their conversation goes something like this:

Friend A: “How about we meet for coffee next Tuesday at 10, at The Red Eye?”  
Friend B: “I could do Tuesday, but not until 11.”  
Friend A: “That works, but I have another meeting at 12:30 on the other side of town, so let’s meet at Common Grounds instead.”  
Friend B: “Okay! See you on Tuesday at 11 at Common Grounds.”  
Friend A: “Sounds good!”  

All of that negotiation takes place over a signaling channel, which is independent of the destination coffee shop. The coffee shop itself can’t work as a signaling channel, because even the question of *which* coffee shop may have to be negotiated—as was the case in the little exchange above.

Establishing a peer-to-peer connection between two browsers works the same way. One browser will make an offer to connect, and the other browser returns some kind of response. The process goes back and forth over the signaling channel, just like friends setting a coffee date over text, until the browsers reach an agreement on how to open the peer connection.

## Using a Lightweight Signaling Channel
What browsers lack and what WebRTC—just like a coffee shop—does not provide is a signaling channel that enables the browsers to negotiate the terms of the connection. The editors of the WebRTC specification have in fact taken great pains to avoid requiring any specific signaling technology: you have to bring your own. So to get WebRTC working, we need to select and set up a signaling channel, probably in the form of a very small server.

For establishing a WebRTC connection, a server-based signaling channel is the most convenient option. In a large-scale application, a server might also do a lot of its usual functions: authenticating and maintaining details for user accounts, and so on. But with WebRTC, a server isn’t strictly necessary. Two peers connecting over WebRTC could, in theory, email or even hand-write and make no-contact delivery of their browsers’ offers and answers to set up the peer connection—just like the two friends could have set their coffee date using semaphore, tin cans on a string, a dead drop, or a whole bunch of other improbable signaling channels.

Even the most basic peer-to-peer application, like the one you’re writing, requires both browsers to visit a URL pointing at some server on the web to download the HTML, CSS, and JavaScript necessary to establish and support the call. With a web server already in the mix, any server-side setup for passing a message from one browser to another would suffice. It would be possible, for example, to write your own signaling server using WebSockets, in any server-side language you choose: PHP, Python, Ruby, and so on.

To save you the trouble, I have written a very small signaling server using Express and Socket.IO. It’s included in the sample code that you already downloaded to work through this chapter. The signaling server works out of the box. While you might find value in experimenting with it, you won’t be writing much server-side code in this book. Our focus is in the browser, because that’s where the real action is.

I do want to take you on a very brief walking tour of this signaling server, so that you better understand the code you’ll be writing for the browser in `main.js`. This server is very little, and very dumb: All it does is shuttle events and messages back and forth. It doesn’t know about WebRTC or anything else that we might be doing. To the greatest extent possible, it’s better to keep knowledge of WebRTC in and between browsers. The signaling server should know and do as little as possible.

That’s why this signaling server component is fewer than a dozen lines, all of which can be reproduced here (you can find the full thing in the `server.js` file). As you’ve seen, sometimes I will walk through setting up a piece of code, line by line. Other times, I will just rip off the band-aid on the whole thing before explaining it. Like here:

```javascript
// server.js
const io = require('socket.io')();
const namespaces = io.of(/^\/[0-9]{7}$/);

namespaces.on('connect', function(socket) {

  const namespace = socket.nsp;
  console.log('Socket namespace: ', namespace.name);

  socket.broadcast.emit('new connected peer');

  socket.on('signal', function({ description, candidate }) {
    socket.broadcast.emit('signal', { description, candidate });
  });

  socket.on('disconnect', function() {
    namespace.emit('new disconnected peer');
  });

});
```

There are two features of this signaling server that matter for the code we’ll write in the browser: a set of events, and a precise 7-digit namespace. Let’s look at the events first.

### The Signaling Channel’s Events
There are six events handled by the signaling server: three that the server listens for (connect, signal, and disconnect) and three that the server emits (new connected peer, signal, and new disconnected peer).

When a peer connects to the signaling server, it will broadcast the `new connected peer` event. When a peer sends a signal to the signaling server, it will rebroadcast the signal—essentially acting as a repeater. And finally, when a peer disconnects from the signaling server, it will emit a `new disconnected peer` event.

That captures everything a basic signaling server needs to do: listen for connections, listen for and repeat signals, and listen for disconnections. That’s it. The code we write in the browser will trigger or respond to each of those events.

### Namespacing the Signaling Channel
The video-chat app you’re building in this chapter is meant to allow multiple different pairs of peers to connect to each other simultaneously and independently. That is no different from how Zoom or Google Meet works. When you go to use Zoom,  you and the person you want to talk to must share (over a signaling channel!) a unique URL like `https://fake-example.zoom.us/j/72072139453`.

The opening lines of the signaling server use a regular expression to verify the structure of a namespace, which is just like a meeting code in Zoom. With that namespace in place, one pair of peers can negotiate their connection over the namespace `/0000001`, while another pair can connect simultaneously over `/0000002`. Their signals will never cross.

Those are very easy to guess patterns, of course, so we’ll use the browser to generate a random number that matches the namespace’s expected pattern. (Later in the book, we’ll use the server to generate namespaces.) If you’ve never worked with regular expressions before, their basic purpose is pretty easy to express: regular expressions describe patterns.

First, let’s add a global variable, `namespace`, on the same line where you previously assigned the `ui` variable to an empty object literal:

```javascript
// main.js
var namespace, ui = {};
```

The namespace doesn’t readily associate with anything else, so that’s why it gets its very own variable. At the very bottom of the `main.js` file, you can create a section for utility functions. Here I’ve written a function called `setOrCheckNamespace` which takes one argument, `hash`, to find the existing URL hash reported by the browser from `window.location.hash`. It then tests that hash against a looking regular expression pattern: `/^[0-9]{7}$/`. Look through the whole thing, and then let’s walk through it together.

```javascript
/*
  Utility functions
*/
function setOrCheckNamespace(hash) {
  var ns = hash.substring(1); // remove the # from the hash
  // Check for a 7-digit namespace...
  if (/^[0-9]{7}$/.test(ns)) {
    console.log('Checked existing namespace', ns);
  } else {
    // ...or create a new 7-digit namespace
    ns = Math.random().toString().substring(2,9);
    console.log('Created new namespace', ns);
  }
  // Return the 7-digit namespace
  return ns;
}
```

The local variable `ns` uses the `substring()` method to remove the `#` (octothorp) that `window.location.hash` always returns. With the octothorp removed, we can check the hash’s value against a regular expression pattern.

Let’s examine the regular-expression pattern `/^[0-9]{7}$/` from the inside out. A pattern that matches exactly seven digits, 0 thru 9, looks like this: `[0-9]{7}`. `[0-9]` represents all numbers in the range zero to nine. The seven in curly braces, `{7}`, means that we want the numbers in the range to appear seven times in a row. The problem is that any string of text that has seven digits in a row would still satisfy the requirement of seven digits.

To make sure the namespace is only ever exactly seven digits and therefore prevent potentially malicious characters from ever reaching the signaling server, the pattern begins with a caret: `^`. The caret is regular-expression symbol used to mark the very beginning of a pattern. A dollar sign `$` marks the very end of the pattern. Without the dollar sign, `0011222-EVIL-BUSINESS-HERE!` would match, because it opens with seven digits (that’s probably why most people find themselves so confused by regular expressions). Bookending the regular expression are slashes, `/`, which demarcate regular expressions in JavaScript, just like single or double quotation marks demarcate strings.

With the regular expression in place, we call the `test()` method on it with the namespace we have on hand, `ns`. If the test fails, either because the hash is mismatched or there’s no hash at all, the function generates a new random hash in a complex one-liner:

```javascript
ns = Math.random().toString().substring(2,9);
```

That generates a random number and converts it to a string. The `substring` method gets called again to take off the first two characters: numbers generated by `Math.random()` always begin with `0.`. The second argument to `substring`, `9`, ensures that we get a seven-digit number, thanks to the first two characters being discarded by the first argument, `2`.

Finally, the function returns the value of `ns`, which will either be the correct hash originally passed into the function, or a brand-new randomly generated hash. Either way, the value will be a seven-digit namespace for connecting to the signaling server.

With `setOrCheckNamespace` function definition in place, you can put it to work at the top of your file:

```javascript
namespace = setOrCheckNamespace(window.location.hash);
window.location.hash = namespace;
```

In case the namespace differs from what `window.location.hash` reported back with, we now set `window.location.hash` to the returned namespace.

Let’s do one more user-facing thing with that `namespace` variable and write a one-off line to set the `h1` text to welcome users to the namespaced room:

```javascript
document.querySelector('#header h1').innerText =
  'Welcome to Room #' + namespace;
```

Reload the page at `https://localhost:3000`. You should see two things: a random, 7-digit hash appended to the URL, something like `https://localhost:3000/#4134710`, and that same hash repeated in the text of the page’s first-level heading: “Welcome to Room #4134710.”

[FIGURE PLACEHOLDER]

Those adjustments are basically cosmetic, changing only the appearance of the URL and page. Now let’s use the `namespace` variable to connect to the the signaling server.

### Automatically Connecting to the Signaling Server
There are a few preliminary steps needed to connect to the signaling server. The first is to return to the `index.html` file, and add another `<script>` tag right above the one that loads the `main.js` file. Point it to the Javascript file, `/socket.io/socket.io.js`, that Socket.IO automatically serves:

```html
<!-- index.html -->
  <script src="/socket.io/socket.io.js"></script>
  <script src="/js/main.js"></script>
</body>
</html>
```

The `main.js` file depends on the contents of the `socket.io.js` file, which is the Socket.IO client, so be sure to list `main.js` second in the HTML.

Back in the `main.js` file itself, let’s connect to the signaling server and output a message to the browser console if the connection is successful and it hears the server emit the `connect` event. You can declare an `sc` variable to hold onto the signaling channel, and then assign its value to connection returned by Socket.IO:

```javascript
var namespace, sc, ui = {};

// Connect to the signaling channel on the namespace
sc = io.connect('/' + namespace);
// Log a success message after connecting to the signaling server
sc.on('connect', function() {
  console.log('Successfully connected to the signaling server!');
});
```

If your browser’s JavaScript console isn’t already open, open it. You should see the success message: “Successfully connected to the signaling server!”

Right now, anyone using your app will automatically connect to the signaling channel simply by loading the page in the browser. That doesn’t give users enough control: we don’t want them joining a call until they’re absolutely ready. That’s why we went to the trouble of building a Join Call button. So let’s put it to work.

### Using the Join Call Button to Connect to the Signaling Server
Earlier, we set up a `handleUiCallButton` function. Let’s call two more functions from within it, `joinCall()` and `leaveCall()`, which can be defined in the utility functions area of your JavaScript file, right alongside the `setOrCheckNamespace` function you wrote a bit ago:

```javascript
// main.js

function handleUiCallButton(event) {
  var callButton = event.target;
  if (callButton.className == 'join') {
    console.log('Joining the call...');
    callButton.className = 'leave';
    callButton.innerText = 'Leave Call';
    joinCall();
  } else {
    console.log('Leaving the call...');
    callButton.className = 'join';
    callButton.innerText = 'Join Call';
    leaveCall();
  }
}

/*
  Utility Functions
*/
function joinCall() {
  sc.open(); // Open the signaling channel
}

function leaveCall() {
  sc.close(); // Close the signaling channel
}
```

There will be more to add to both the `joinCall` and `leaveCall` functions. But to get started, they will be responsible only for opening and closing the signaling channel, which provides its own `open()` and `close()` methods. We just need to call them.

Now that your JavaScript is set to manually open and close the connection to the signaling server, you can modify how the signaling channel is configured by passing in an options object that sets `autoconnect` to `false`:

```javascript
// main.js
sc = io.connect('/' + namespace, { autoConnect: false });
```

Reload the page in your browser again. You shouldn’t see the “Successfully connected to the signaling server!” in the JavaScript console until you click the Join Call button. If you click the button a few more times, joining and leaving the call, you’ll see the success message each time you click Join Call. Nice! Because signaling channel is powered by Socket.IO, it has a property, `active`, which returns `true` when the signaling server is connected, and `false` when it’s not. If you like, hit the Join Call button and then type `sc.active` and hit Return in the browser’s JavaScript console. It should report `true`. Hit the Leave Call button, type `sc.active` again, and it should report `false`. That’s a good way to verify that your button really is opening and closing the signaling channel.

### Setting Up the Remaining Signaling Channel Callbacks
One more thing, and then we’re going to take a short break from signaling channel stuff. Let’s set up callbacks for all three of the events from the signaling channel, and put placeholder function definitions for them in a new Signaling Channel Callbacks area of the JavaScript file. The anonymous callback function that logs “Successfully connected…” can also be rewritten as a named callback function, in case there’s more to do with that event in the future. That will make for four tidy lines of signaling channel events and a separate, organized place in the file for their callback definitions:

```javascript
// main.js
// Signaling channel events and their callbacks
sc.on('connect', handleScConnect);
sc.on('new connected peer', handleScNewConnectedPeer);
sc.on('new disconnected peer', handleScNewDisconnectedPeer);
sc.on('signal', handleScSignal);

/*
  Signaling-channel callback functions
*/
function handleScConnect() {
  // Log a success message after connecting to the signaling server
  console.log('Successfully connected to the signaling server!');
}
function handleScNewConnectedPeer() {
  // TODO: Handle a newly connected peer
}
function handleScNewDisconnectedPeer() {
  // TODO: Handle a disconnected peer
}
function handleScSignal() {
  // TODO: Handle anything that comes across as a signal
}
```

There are some additional modifications to those function definitions on the way, but for now they’re enough to prevent the browser from complaining about missing references to them in the stack of `sc.on` events.

## User Media: Requesting Mic and Camera Permissions
Let’s turn to handling a task that’s pretty awesome and fun: requesting user permission for the browser to use the camera (and mic), and displaying live video in the HTML’s self video element.

To start, add another variable to the growing list of variables at the top of the `main.js` file: this will be called `self`, and should also be set as an empty object literal:

```javascript
// main.js
var namespace, sc, self = {}, ui = {};
```

Back down in the utility functions area of your JavaScript file, define a new function called `requestUserMedia`. This time, though, you’ll need to preface the `function` keyword with `async`. I’ll explain that along with the rest of the function in a moment, but I promised fun first. So call this function and set up its definition like so, and then reload the page in your browser:

```javascript
// main.js
requestUserMedia();

/*
  Utility Functions
*/
async function requestUserMedia() {
  var media_constraints = { video: true, audio: false };
  var local_stream = new MediaStream();
  self.media = await navigator.mediaDevices.getUserMedia(media_constraints);
  local_stream.addTrack(self.media.getTracks()[0]);
  // TODO: do this in reference to the `ui` object literal?
  document.querySelector('#self').srcObject = local_stream;
}
```

When you reload the page, you should be greeted with your browser’s dialog box for media permissions. Figure 2.X shows the dialog box in Firefox:

[FIGURE PLACEHOLDER]

Be sure to click Allow on the permissions dialog. While you can opt to let the browser remember your decision, I like to always be prompted when I’m developing something for WebRTC—just to remind myself of the flow of the interface for first-time users. Once you click allow, after a beat you should see your very own face, streaming in real time, right on an otherwise static HTML page.

[FIGURE PLACEHOLDER]

All right. So what’s happening with that `requestUserMedia` function?

Let’s start by looking inside the body of the function. There are two variables, `media_constraints` and `local_stream`. The media constraints are very basic: it’s an object literal that says the app wants access to user video, but not audio. Again, we’re disabling audio so that your computer doesn’t go full Jimi Hendrix on the feedback. Later in the book, you will apply more constraints to the requested user media, but this is all that’s needed for now. The local stream is set to a new `MediaStream` object, which will hold onto the tracks returned by the user’s devices. The relationship between streams and tracks is a little complicated, but for now it’s enough to think of a stream as a container for tracks. Tracks represent the actual audio and video data returned from a user’s camera and microphone.

Below those lines, the function calls up the global `self` object, and sets a property on it called `media`. That property will hold onto whatever is returned from the `navigator.mediaDevices.getUserMedia()` function once a user responds to the permissions dialog. More in a moment on what that `await` keyword is doing.

Once `self.media` is set, the function adds media tracks to the `local_stream` using the `addTrack()` method on the `MediaStream` object. To do that, the function calls the `getTracks()` method on the `self.media` object. Ordinarily `getTracks()` returns an array of tracks returned by user devices; the little `[0]` tacked onto the end means we’re just interested in the very first track.

Once the tracks have been added to `local_stream`, the function uses `document.querySelector` to select the self video element from the DOM and set its `srcObject` property—that is, the video source for the element to play—to `local_stream`. As you’ll see, the method for setting the peer’s `srcObject` is almost the same.

### Async, Await, Say What?
Stepping outside the function now: there’s that curious `async` keyword hanging out in front of the more familiar `function` keyword. That’s used to define the `requestUserMedia` function so it can make use of a related keyword, `await`.

So far in this chapter, we’ve been dealing with an old-school JavaScript pattern for handling asynchronous code: callbacks, usually tied to events. The Join Call click event uses this pattern, as do all of the signaling-channel events we set up in the last section. In pseudo code:

```javascript
doSomethingAsyc('then', doSomethingElse);
```

As a language, JavaScript has evolved to handle more complex cases of asynchronous code. The callback pattern looks pretty good in that simple case above, but it lead to “pyramids of doom”, with callbacks inside of callbacks inside of callbacks. JavaScript’s evolution to address that problem came in the form of something called the `Promise` object. We don’t need to go too deep on that here, but what promises did was to make callback-based code a little more manageable:

```javascript
doSomethingAsync()
  .then(doSomethingElse)
  .then(doSomethingElseEntirely);
```

The `async` and `await` keywords transform that pseudocode to look a little more familiar to those of us used to functional programming in JavaScript. Functional programming is what is what we’ve been doing this whole chapter. Instead of chaining together all of those `.then()` calls, we can use `async` and `await` instead (which also requires us to handle the `data` these imaginary async functions return):

```javascript
doSomethingElse(await doSomethingAsync());

async function doSomethingElse(data) {
  doSomethingElseEntirely(data);
}
```

In that example, `await` appears right where the `data` argument gets passed into the `doSomethingElse` function. But it can also appear in the body of the function:

```javascript
async function doSomethingElse(doSomethingAsyncByReference) {
  var data = await doSomethingAsyncByReference();
  doSomethingElseEntirely(data);
}
```

In that case, `doSomethingElseEntirely` will not be called until the data has been returned from `somethingAsync()`.

So why do we care about `async` and `await` in the context of media permissions? Well, it turns out that the user-media API we need to use for the fun stuff, `navigator.mediaDevices.getUserMedia()`, is promise-based and asynchronous. That means that even while the media permissions dialog is showing, users can continue to interact with the page: try clicking the Join Call button while the permissions dialog box is open. The button keeps working, no problem (which might actually be a problem—later in the book you will experiment with refining the behavior of the Join Call button while awaiting the user to respond to the media permissions dialog).

The `requestUserMedia` function you’ve written will not move ahead until it has the permissions and stream from the user’s media. Without `await`, the `getTracks()` method would be called on `self.media`  too soon, before it has any data. That would throw an error in JavaScript and halt the execution of the entire app right then and there.

All right. We’ve got the UI prepared, the signaling channel set up, and now we’ve got streaming video set up on the self side of the connection. Now it’s time to bring it on home, and get this peer connection set up and established.

## Setting Up the Peer Connection
Here we go. Let’s kick things off by adding two more variables to the growing  stack at the top of `main.js`: `rtc_config` and `peer`, which like `self` will be another object literal:

```javascript
// main.js
var namespace, sc, rtc_config, peer = {}, self = {}, ui = {};
```

Eventually, `rtc_config` will include some important configuration values. But for testing it locally, setting it to `null` explicitly tells the peer connection that the configuration has no value. Immediately after that, we’ll use the `peer` object to hold onto a new RTC peer connection object:

```javascript
// main.js
// Set up the peer connection
rtc_config = null;
peer.connection = new RTCPeerConnection(rtc_config);
```

And that’s it!

Okay, not really. Not by a long shot. There’s much more to come. Those two lines of code are accomplishing a lot, though. The `peer.connection` property now holds a new RTC peer connection object, which is the key piece of everything left to build.

But before we leave the enchanted land of variable assignment for the kingdom of callback function definitions, let’s set a few more properties on `self`:

```javascript
// main.js
// Set up self variables
self.isPolite = false;
self.isMakingOffer = false;
self.isIgnoringOffer = false;
self.isSettingRemoteAnswerPending = false;
```

We will return to all of those in just a bit. The one that is most interesting for the time being is the first one: `self.isPolite`, which is initially set to `false`. That means we’ve just admitted to the world that `self` is surly and rude. But why?

### Symmetric Code, Asymmetric Function
There are many things about WebRTC that are a little mind-bending. The source of those mind-benders is having the same exact codebase—HTML, CSS, and JavaScript—loaded in the browsers on both ends of a call. That means the code is symmetric: it’s the same in each browser. There’s no need to goof around writing server-side logic that sends different code to different peers. Anyone on the call is just a joiner.

Symmetric code, like the symmetric joiner interface it supports, provides both peers (and eventually, in Chapter Five, *all* peers) with the ability to initiate and answer a call.

“But wait,” you might say. “Why are we suddenly talking about initiating and answering calls? Didn’t we explicitly set up a joiner pattern, rather than a caller pattern?” Yes, yes we did. The only real piece of UI in our interface—the Join Call button—means that we aren’t going to worry at all about who’s calling whom. The peers just join the call.

But just because *we’re* not going to worry about the caller and answerer roles doesn’t mean that WebRTC won’t. In fact, it worries quite a lot about that. The object created by `new RTCPeerConnection` that we’re hanging onto in `peer.connection` is basically a giant worry-wart. And like most worry-warts, it doesn’t quite know what to do with all its worries: it will freak out unless, almost like good therapists, we do a really good job giving it coping strategies for managing its worries.

That brings us back to the `self.isPolite` value: it represents the core technique for establishing a worry-free WebRTC connection. In each group of two peers connecting over our app, one peer will be polite (`self.isPolite = true`), and one will be impolite (`self.isPolite = false`). So even though the same code appears in both peers’ browsers, each browser will behave a little differently, being either polite or impolite, depending on the circumstances of the call. And that’s how we will be able to pull off asymmetric behavior using symmetric code.

### Gaining Acceptance in Polite Society
That all might sound good. But if the code is still *exactly* the same in both browsers, what sorcery is required to make sure that one peer is polite and one peer is impolite?

We could do a lot of things to determine that. We could write a function that does a virtual coin-toss to determine which peer is polite. We could make it weird and administer a short personality test to figure out which peer really *is* more rude in real life. Or we could do what I do, and just let the order of joining the call do all the work for us. The first person to join the call will be polite, and the second person to join will be impolite. Mathematically speaking, there is an extremely low chance that both peers could join the at exactly the same fraction of a second. But it’s so improbable that we don’t even have to worry about it.

Recall that the signaling server does a broadcast-emit of a `new connected peer` event. The broadcast emit is special, because it goes to everyone on the namespace *except* the person who triggered the event. Every time someone joins the signaling channel, `new connected peer` fires. You’ve already written a callback in the `main.js` file in the browser to handle it:

```javascript
// main.js
sc.on('new connected peer', handleScNewConnectedPeer);
```

So open up that empty `handleScNewConnectedPeer` function definition, and add just a single line to it that sets `self.isPolite` to `true`:

```javascript
// main.js
function handleScNewConnectedPeer() {
  self.isPolite = true;
}
```

Here again we appear to end up with the same symmetric code problem: both peers join the call, meaning both peers will trigger the signaling server to broadcast the `new connected peer` event. That’s true.

The trick is that when the first peer joins the call, we’re in Zen koan territory: *If a tree falls in the forest and no one is around to hear it, does it make a sound?* When the first peer triggers the `new connected peer` broadcast event, *there’s no one around to receive it*. The second peer hasn’t connected yet, and likely will not connect in the nanosecond timeframe required to accidentally receive it.

But: When the *second* peer connects, `new connected peer` fires again—only this time, the first peer *is* around to receive it. And so the first peer becomes the polite one (`self.isPolite = true`), while the second peer stays the impolite one (`self.isPolite = false`, as initially set near the top of the `main.js` file). Even though the second peer also has the `handleScNewConnectedPeer` function available, it will never fire, because the second peer will never hear a `new connected peer` event.

### You Can Soon Add “Event Planner” to Your Resume
Hold onto the fact that one peer on a call is polite, and the other is impolite, while you stack up yet another collection of events in your JavaScript. These events go inside the same `joinCall` function that you set up to connect the signaling channel:

```javascript
// main.js
function joinCall(clickEvent) {
  sc.open(); // Open the signaling channel

  // Register RTCPeerConnection events and their callbacks
  peer.connection.onnegotiationneeded = handleRtcConnectionNegotiation;
  peer.connection.onicecandidate = handleRtcIceCandidate;
  peer.connection.ontrack = handleRtcPeerTrack;

  // TODO: Add self.media to the peer connection
}
```

Those events use different syntax for registering callbacks from what we’ve seen so far. Because these three events—`negotiationneeded`, `icecandidate`, and `track`—are so commonly used with the `RTCPeerConnection` object, it gives us shorthand properties prefixed with `on`. You might have seen this before with the DOM: you can use `domObj.addEventListener('click',handleClick)` or the shorthand `domObj.onclick = handleClick`. Both syntaxes achieve the same thing.

Open up one final area toward the bottom of your `main.js` file, and label it something like  “RTCPeerConnection callback functions“ in a JavaScript comment. Then drop in three new function definitions for the events you registered them on above:

```javascript
/*
  RTCPeerConnection callback functions
*/
async function handleRtcConnectionNegotiation() {
  // TODO: Handle connection negotiation
}
function handleRtcIceCandidate() {
  // TODO: Handle ICE candidate
}
function handleRtcPeerTrack() {
  // TODO: Handle peer media tracks
}
```

Because each of these functions are registered within the `joinCall` function, they will begin to go to work immediately. The first two functions are responsible for sending offers and connectivity candidates to the other peer. The third is for media tracks, which will get wired up later.

The first function, `handleRtcConnectionNegotiation`, will use `await` in its body, so we prefix it with `async`, just like the media-permissions utility function. What it’s awaiting is a promise-based RTC peer connection method, `setLocalDescription()`, which prepares an offer based on the conditions on the self side of the call. While the function awaits that local description, `self.isMakingOffer` is set to `true`. Once the offer—`peer.connection.localDescription`—has been sent over the signaling channel, the function is no longer making an offer, so `self.isMakingOffer` gets set back to its initial state, `false`.

Now don’t be fooled by `await`, in this function or anywhere else: under most conditions, the wait is less than a blink of an eye. When we get to handling offers (and answers) as they come in, you’ll see that blinks of an eye can still matter—especially to a worry-wart RTC peer connection.

```javascript
async function handleRtcConnectionNegotiation() {
  self.isMakingOffer = true;
  await peer.connection.setLocalDescription();
  sc.emit('signal', { description: peer.connection.localDescription });
  self.isMakingOffer = false;
}
```

While that function handles offer descriptions, the second callback handles ICE candidates. I know, that name in all caps makes me think of unpleasant things, too. But ICE in this case stands for *Interactive Connectivity Establishment*, an IETF-defined protocol that enables one browser to determine how to connect to another over the internet or a local network (https://tools.ietf.org/html/rfc8445). While `localDescription` describes the media that will flow over the connection, ICE candidates describe possible routes over the network to establish the connection.

The `handleRtcIceCandiate` callback not terribly involved. When a candidate becomes available, the browser just needs to send it over the signaling channel, attached again to the `signal` event.

```javascript
function handleRtcIceCandidate({candidate}) {
  sc.emit('signal', { candidate: candidate });
}
```

### Destructuring Objects
Something might look odd to you in that function definition: the `{candidate}` in curly braces that gets passed into the function as an argument. Ordinarily, when we see curly braces, they look more like what gets emitted on the signaling channel: `{ candidate: candidate }`, with one value and another separated by a colon, `:`. Straight-up object literals.

A newer feature available in JavaScript syntax is something called *destructuring assignment*. It’s a shorthand way of pulling a value out of an object literal and perhaps assigning it to a freestanding variable. Here’s a simple example:

```javascript
var obj = { one: 1, two: 2 }
```

If we were interested in assigning the value of `one` to its own variable, we could write this:

```javascript
var one = obj.one;
console.log(one); //-> 1
```

But restructuring assignment allows us to write this instead:

```javascript
var {one} = obj;
console.log(one); //-> 1
```

In short, the value in the curly braces gets assigned as a standalone representation of whatever matching property exists in the object—assuming the property exits. If the property doesn’t exist, the value is undefined:

```javascript
var {three} = obj;
console.log(three); //-> undefined
```

So what `function handleRtcIceCandidate({candidate})` does is extract just the value of `candidate` from the chunk of data returned by `peer.connection.onicecandidate`.

## The “Perfect Negotiation” Pattern
With the `handleRtcConnectionNegotiation` and the `handleRtcIceCandidate` callback functions defined, you now have most of the sending side of the connection in place. The big piece remaining is the receiving side to handle incoming descriptions and ICE candidates.

We’re going to build that logic according to the “perfect negotiation” pattern described in the WebRTC specification:

> This pattern has advantages over one side always being the offerer, as it lets applications operate on both peer connection objects simultaneously without risk of glare (an offer coming in outside of “stable” state). The rest of the application may use any and all modification methods and attributes, without worrying about signaling state races. (https://www.w3.org/TR/webrtc/#perfect-negotiation-example)

That’s a dense quotation, but here’s the essence: in a symmetrical app like the one you’re building, both sides of the connection can pass an offer to the other. That’s essential because, with a joiner app, we don’t need to know or care who joins first.

The catch is that when one side of a peer connection sends an offer description, it expects an answer in response. In fact, under typical conditions it won’t respond to anything *but* an answer. If both sides of a connection have sent offers out, they basically end up in a stalemate—what the WebRTC spec calls “glare.” It’s a lot like a face-to-face conversation when two people start speaking at the same time. Assuming they don’t escalate into a shouting match with each other, one person in such a conversation will usually say “Oh, I’m sorry—you go ahead.” That’s conceptually identical to the behavior of the polite peer in establishing the peer-to-peer connection.

All of the perfect-negotiation logic will appear inside the `handleScSignal` callback, one of the functions you defined earlier. Let’s open that function definition up, make it `async`, and pass in the data received over the signaling channel. Inside the function, set up an if/else statement that will kick into action depending on whether it’s a description or ICE candidate that’s been received:

```javascript
// main.js
async function handleScSignal({candidate, description}) {
  if (description) {
    // Work with an incoming description (offer/answer)
  } else if (candidate) {
    // Work with an incoming ICE candidate
  }
}
```

The function is set up to act based on whether it’s receiving a description or a candidate, which we conveniently access using destructuring assignment again, this time with two values separate by a comma: `{candidate, description})`. If it’s a candidate coming across, `description` will be `undefined` (and therefore `false` in the `if` statement). If it’s a description, `candidate` will be undefined.

### Determining Whether to Ignore Offer Descriptions
To correctly handle descriptions, or just ignore them entirely, we need to work through an unavoidably dense set of true/false conditions to get to that “Oh, I’m sorry—you go ahead” state in peer-to-peer connection negotiation. Let’s rip off another band-aid, and then talk it through:

```javascript
// main.js
async function handleScSignal({candidate, description}) {
  if (description) {
    // readyForOffer is TRUE if the client is not making an offer,
    // AND EITHER the signaling state is 'stable'
    // OR setting the remote offer is pending
    var readyForOffer =
          !self.isMakingOffer &&
          (peer.connection.signalingState == "stable" || self.isSettingRemoteAnswerPending);

    // offerCollision is TRUE if we've received an offer AND we're not ready for an offer
    var offerCollision = description.type == "offer" && !readyForOffer;

    // ignore offers if impolite and there's an offer collision
    self.isIgnoringOffer = !self.isPolite && offerCollision;
    // and if we are ignoring offers, let's get out of the handleScSignal callback immediately:
    if (self.isIgnoringOffer) {
      return;
    }
  } else if (candidate) {
    // Work with an incoming ICE candidate
  }
}
```

That’s a lot of code. What it’s doing at every turn is looking for true and false conditions. It actually reads a little more sensibly by starting at the bottom, with the `if (self.isIgnoringOffer)` block, and working back up to the top. What that `if` statement does is decide whether to ignore an incoming offer. If it does, that one-line `return;` has the effect of exiting the `handleScSignal` function without doing anything else.

But to determine whether `self` is ignoring offers, we need this line, which assigns either a `true` or `false` value to `self.isIgnoringOffer`:

```javascript
self.isIgnoringOffer = !self.isPolite && offerCollision;
```

What that means is that `self` will ignore offers if it’s impolite (`!self.isPolite == true`) and (`&&`) there’s an offer collision (`offerCollision == true)` An impolite peer will have `self.isPolite == false`, so adding an exclamation mark in front of `self.isPolite` means *not.* I know, it’s confusing. I find it helpful to say “not” aloud when reading through Boolean-happy code like this.

In programming terms, recall that the exclamation mark returns the *opposite* value of `self.isPolite`. So stick with it: with the exclamation point, the impolite peer will evaluate as `true` here, and the polite peer will evaluate as `false`.

The polite peer will never ignore offers: `false` on one side of an AND expression (`&&`) will always make the whole expression false. That’s the polite peer saying, “Oh, I’m sorry—you go ahead” in the face of an offer collision.

Determining whether there is an offer collision, then, is the other piece of information needed to set the `self.isIgnoringOffer` state to either true or false. The value of `offerCollision` will be either true or false based on this line above `self.isIgnoringOffer`:

```javascript
var offerCollision = description.type == "offer" && !readyForOffer;
```

This will assign a value of `true` to `offerCollision` if two things are true: 1) the incoming `description.type` is `offer` (not `answer`) and 2) the `self` side of the call is not ready to receive an offer: `!readyForOffer`. The syntax looks a little wild, but just remember that a single `=` sign is for assignment—giving `offerCollision` a value—and a double `==` is a comparison operator: is `description.type` equal to `"offer"`? If that is true, and the peer is not ready to receive an offer, then there’s an offer collision.

And that takes us to the top line of this mountainous climb, where there’s one final tangle of Boolean values to straighten out:

```javascript
var readyForOffer =
  !self.isMakingOffer &&
    (peer.connection.signalingState == "stable" ||
      self.isSettingRemoteAnswerPending);
```

Let’s break this one down, too. The `self` side of the connection will be ready for an offer (`readyForOffer == true`) if it is not making an offer of its own and if at least one of two other things are true: either 1) the peer-connection’s signaling state is `"stable"` or 2) the `self` side of the connection is trying to set a remote answer. We will investigate the possible values of the peer connection object’s `signalingState` later in the book; for now, let’s just accept that one of its possible states is `"stable"`.

The other two values—`self.isMakingOffer` and `self.isSettingRemoteAnswerPending` are determined elsewhere in the connection logic. You’ve already seen the first one: you set `self.isMakingOffer` in the `handleRtcConnectionNegotiation` callback. To refresh your memory, here’s that connection-negotiation callback function again:

```javascript
// main.js
async function handleRtcConnectionNegotiation() {
  self.isMakingOffer = true;
  await peer.connection.setLocalDescription();
  sc.emit('signal', { description: peer.connection.localDescription });
  self.isMakingOffer = false;
}
```

The code is the same as what you already wrote, but now there’s something new worth pointing out here. The first peer to join the call already fires a signaling-channel event into the ether: the `new connected peer` event. But the first peer will also almost certainly fire its first offer into the ether, too: the other peer will not be around to receive that initial offer from over the signaling channel.

And that’s why it’s smart to set the first-connecting peer as the polite one: by the time the second peer joins the call, the first is quietly awaiting an answer that will never come. And instead of sending the answer that the first peer expects, the second peer fires off an offer of its own. That’s glare: both sides having offers out, and both sides expecting answers that *ordinarily* would never come. But because the first peer is polite, it will ditch its own initial offer (a process called *rollback*) and immediately respond to the second peer’s offer with an answer. Let’s write the code for that next.

### Handling Offer and Answer Descriptions
Whether the incoming description is an offer or an answer, it gets passed to the `setRemoteDescription` method provided by `RTCPeerConnection`. Add this code to the bottom of the `if (description){ }` block inside of the `handleScSignal` callback:

```javascript
// main.js
    // inside the handleScSignal() function definition

    // If we're working with an answer,
    // `isSettingRemoteAnswerPending` is true
    self.isSettingRemoteAnswerPending = description.type == "answer";
    await peer.connection.setRemoteDescription(description);
    self.isSettingRemoteAnswerPending = false;

```

That’s the code that sets `self.isSettingRemoteAnswerPending` to `true`, for use in determining whether `readyForOffer` is true or false. It’s only set when the `description.type` is `"answer"`. The description received, whether an offer or an answer, then gets passed into the `setRemoteDescription()` method on the peer connection. The `setRemoteDescription` method is async, so it’s necessary for the function to `await` its completion. `self.isSettingRemoteAnswerPending` gets set back to `false` at that point, assuming it was set to `true` to begin with.

Below that, let’s add one final block to the description-handling logic. If what’s come in is an offer, not an answer, it’s necessary to answer it:

```javascript
    // inside the handleScSignal() function definition

    if (description.type == 'offer') {
      // Generate the answer...
      await peer.connection.setLocalDescription();
      // ...then send it back over the signaling channel:
      sc.emit('signal', { description:
         peer.connection.localDescription });
    }
```

This little block of code checks to see if the description is an offer. The description will have already been added to `setRemoteDescription` on the lines above, so this call to `setLocalDescription` responds behind the scenes to whatever was passed in to `setRemoteDescription`. After `setLocalDescription` has done its work, the result gets passed back over the signaling channel, care of the `localDescription` property on the peer connection object.

Putting it all together, the logic for handling incoming descriptions now looks like this:

```javascript
// main.js
async function handleScSignal({candidate, description }) {
  if (description) {
    var readyForOffer =
          !self.isMakingOffer &&
          (peer.connection.signalingState == "stable" || self.isSettingRemoteAnswerPending);
    var offerCollision = description.type == "offer" && !readyForOffer;
    self.isIgnoringOffer = !self.isPolite && offerCollision;

    if (self.isIgnoringOffer) {
      return; // exit the function if ignoring offers
    }

    self.isSettingRemoteAnswerPending = description.type == "answer";
    await peer.connection.setRemoteDescription(description);
    self.isSettingRemoteAnswerPending = false;

    if (description.type == 'offer') {
      await peer.connection.setLocalDescription();
      sc.emit('signal', { description:
        peer.connection.localDescription });
    }
  // end handling of description signals;
  } else if (candidate) {

  }
}
```

That’s a lot of complex code you’ve written. The good news is that it’s also completely portable: you can basically reuse this in any WebRTC app that makes use of perfect negotiation, as you’ll see in all the other apps built in this book. This code is meant to work independently of any other application logic you find yourself writing. Let’s finish the job and handle the incoming ICE candidates.

### Handling Incoming ICE Candidates
Handling the ICE candidates sent over the signaling channel is way less complicated than handling descriptions. Here’s the complete `else if (candidate)` logic:

```javascript
// main.js

  // inside the handleScSignal() function definition
  } else if (candidate) {
    // Ignore candidates sending empty candidates (Safari freaks out)
    if (candidate.candidate.length > 1) {
      // Add the ICE candidate to the peer connection
      await peer.connection.addIceCandidate(candidate);
    }
  }
```

I’ve added a piece of protective logic here — the if statement tests the `length` property on the candidate string to make sure it’s greater than one. Some browsers occasionally burp up empty ICE candidates, which ordinarily isn’t a problem—except that Safari and some older browsers are highly offended by them. So assuming the `candidate.candidate` value isn’t empty, we pass the entire `candidate` signal into the `addIceCandidate` method on the RTC peer connection.

And that, I’m delighted to tell you, is that. All of the negotiation logic is in place. Two more short pieces of code, and you’ll be all set to test this thing out.

## Sending and Receiving Media Tracks
All that’s left is to handle the media tracks. While you’ve already set your local stream to appear on the self `<video>` element, you need to also add that stream to the peer connection so that the remote peer can view it:

```javascript
// main.js
// bottom of the joinCall function definition

  // Add self media to the peer connection:
  for (var track of self.stream.getTracks()) {
    peer.connection.addTrack(track, self.stream);
  }
```

A `for` loop races through all of the tracks in your `self.stream`, and attaches each one to the peer connection, which has its own `addTrack()` method. WebRTC will happily manage a stream for us, so `self.stream` gets added to the connection, too. Those three lines are all that’s needed to add a stream on the self side of the call and transmit it.

On the receiving side, we need logic that’s going to listen for the remote peer’s media coming in over the connection and then set the peer `<video>`’s `srcObject` to the incoming stream. To do this, return to the `handleRtcPeerTrack` RTC callback function definition you created earlier:

```javascript
function handleRtcPeerTrack({track, streams: [stream]}) {
  track.onunmute = function() {
    document.querySelector('#peer').srcObject = stream;
  }
}
```

Once more we see destructuring assignment at work: this time, it pulls out the returned track and stream from the `onaddtrack` event handler on the `RTCPeerConnection` object. In the body of that function, another event handler, `onunmute`, will set the `srcObject` to the incoming stream.

I can’t think of a more poorly named event in the wide world of Web APIs than `unmute` on `MediaStream`: `unmute` as well as `mute` has nothing to do with audio in this context. Both events are defined in the draft specification for the Media Capture specification (https://w3c.github.io/mediacapture-main/#event-mediastreamtrack-unmute). Here’s what each of these events mean, and when they fire:

* A `mute` event fires when a media-stream track “is temporarily unable to provide data”
* An `unmute` event when a media-stream track “is live again after having been temporarily unable to provide data”

So `onunmute` on a track basically means “data from this track has started flowing across the connection from the remote peer.” When that data starts to flow, go ahead and attach the entire stream to the `srcObject` on the peer video.

## Testing It All Out
Now it’s time to test it all out. Reduce the size of the browser window you’ve been working in so there’s room to open a second just like it. Be sure to copy and paste the URL from the first window into the second, so that you’re communicating on the same signaling channel. Refresh both windows, grant permissions for the camera, and then hit Join Call in one window, then the other. Then get ready to see a whole lot of your own face on the screen.

[FIGURE PLACEHOLDER]

If things aren’t working, don’t lose heart. Check your console for any errors, including the ones from the calls to `console.log` that you sprinkled in along the way. You might also want to check your work against the completed example at https://github.com/prag-webrtc/ch-2-complete/.

Once it’s all working, congratulate yourself. This is a huge feat, and you’ve now mastered all of the essential techniques for establishing peer-to-peer connections with WebRTC. You’re streaming live video like a pro. As cool as that is, you can use a peer-to-peer connection to stream more than just audio and video. The next chapter will look at opening up pipelines for streaming all kinds of data over the `RTCDataChannel` interface, independent of whether an app includes streaming media or not.
