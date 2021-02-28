Dr. Karl Stolley
karl.stolley@gmail.com  
312-351-4326  
4134 W Wellington Ave  
Chicago, IL 60641  
https://stolley.co/

# WebRTC
## Build Real-Time Web Applications in the Browser


## Bio
I’m a tenured associate professor of information technology and management at Illinois Institute of Technology in Chicago. I’ve been teaching, researching, and writing about web development up and down the stack since I was in graduate school, which means I’ve been at this for about twenty years. I’m a frequent speaker at national and international academic conferences, and a presenter and active participant in several professional developer groups and meetups in Chicago, and I’m a co-organizer of the Chicago NodeJS meetup group. I have authored numerous academic and industry articles, book chapters, and one book, which sold well enough to go into a second edition. I find time to moonlight as a web designer and developer and have completed substantial work for the Purdue Online Writing Lab (years ago, before it turned to evil), the Newberry Library, and a variety of smaller organizations, businesses, and academic projects you’ve never heard of. I am a regular contributor to the copy, example code, and browser compatibility data at Mozilla Developer Network— which is just one example of my passion for reading, referencing, and contributing to documentation.

I’m finishing up this book proposal on WebRTC just weeks after teaching a successful sixteen-week course in Web Real-Time Communications to students specializing in web design and development (see the course at https://courses.stolley.co/rtc/). The all-online pandemic semester was a challenging nightmare that nobody wanted, but it provided an ideal backdrop for teaching to the WebRTC spec and demonstrating the astonishing possibilities of streaming media and arbitrary data directly from one web browser to another.

The biggest challenge I faced in putting together my WebRTC class over the last year was the lack of a suitable, up-to-date book for teaching or engaging in self-study of WebRTC—especially in the context of web development. As I detail in the competing books section below, there are a few rapidly aging titles on the market that are now little more than works of history. There is little, in book form or otherwise, that offers a substantial, pragmatic application of the WebRTC specification as it has matured since achieving Candidate Recommendation status in late 2017. (The spec became a full W3C Recommendation on January 26, 2021, while I was preparing this proposal).

So with no current book that I could assign without feeling professionally negligent, I set out to gather and assess as many article-length resources and tutorials on WebRTC as I could. I assigned the best of those to students, complete with supporting notes, numerous corrections, and the earliest fragments of this proposed book. But it wasn’t just explanatory material that I found lacking: there were very few well written code examples in the wild that I could responsibly teach from. Even Google Codelab’s semi-famous *Real time communication with WebRTC* tutorial has some fatal errors, on top of being a rather shoddy code base generally—which is a shocking thing to say about Google. But it’s true. So I created hundreds of lines of example code from scratch to support all of the brand-new material I prepared on WebRTC (lectures, demonstrations, written guides and instructions).

And that’s perhaps my most compelling relationship to WebRTC in regards to this book proposal, and what I hope will help you see me as an author worthy of the subject and of Pragmatic Programmers: I’ve taught this material, at a distance and through a lot of writing, to a group of eager but not super prepared students, many of whom are already working professionals. That means that the bulk of the material and code that will make up this book has already been road-tested with a group of people who are not all that different from Pragmatic’s developer audience. I’ve gotten to see first hand what works and what kinds of questions and problems come up, so I can head them off in advance. I know this material well from both the research and the applied sides. I’m conversant with the WebRTC specification itself, and I have a good sense of how to effectively convey this material to the developer audience I want the book to target.

In short, I’m excited to condense, refine, and extend all of that material into a book—not as a teacher writing to students, but as a professional writing to other professionals. And not just any book, either: I want to write a book that reads the way Pragmatic books are written, and learn how to produce it the way Pragmatic books are produced. (I’ve previously served as a reviewer on a few Pragmatic titles, including Trevor Burnham’s *Async Javascript*. Trevor was cool enough to mention me by name in his book’s acknowledgements, and I’ve been searching for years to find the right topic to write a Pragmatic book of my own. With WebRTC, I think I’ve found a winning topic that will add real value to Pragmatic’s catalog.)

## Overview
My proposed book presents a thorough, up-to-date treatment of WebRTC—a full W3C Recommendation as of January 2021—as implemented in modern web browsers. The book I am proposing is one that I wish someone else had already written in Pragmatic’s hallmark approachable style and no-nonsense length, so I could have assigned it to students after having read it myself—as I have done with so many other Pragmatic books (this semester alone, students in my Web Systems Integration class are reading three different Pragmatic titles).

The book is structured around several small but complete applications that leverage WebRTC. Unlike the aging books currently on the market, my book will follow the Pragmatic Programmers tradition of getting right down to work. There are no long chapters offering ponderous histories and theories behind WebRTC (the market-leading book, which I describe below, doesn’t get to building anything until its penultimate chapter). Instead, by the end of the second chapter, readers will have built a simple but complete video-streaming application that works directly peer to peer.

My book will focus exclusively on WebRTC as implemented in modern browsers: while it will make mention of the WebRTC specification when doing so helps to bolster a programming concept, the bulk of the book is focused on WebRTC APIs in browsers, especially the `RTCPeerConnection` and `RTCDataChannel` interfaces. It will guide readers to use only the native APIs, rather than any third-party libraries—many of them of dubious quality—that have been popping up in recent years.

The emphasis of the book is on getting down to building real applications with WebRTC’s browser APIs. Each chapter will feature at least one small but complete application, which I have specifically designed to draw attention to unique and interesting aspects of WebRTC in the browser--and which I will help abstract for readers so that they can extend what they’re learning to build novel and exciting WebRTC applications of their own.

Because the book will be focused on WebRTC’s implementation in the browser, it includes very little work on the server side, which may come as a surprise. WebRTC does require some kind of signaling channel in order to establish a peer connection. I have already built a small but feature-complete signaling server using ExpressJS and Socket.io. Readers will walk through the features of the signaling server I have built, and I will encourage them to experiment and extend it to meet their own needs. A chapter at the end of the book will look at deploying WebRTC applications, including setting up STUN and TURN servers to facilitate network traversal and data relay under firewalled network conditions that disallow direct peer-to-peer connections. That is a concern only in approximately 10% of peer-to-peer connections, usually involving wildly restrictive corporate or government firewalls.

Although each chapter will include a certain amount of coverage of interface design problems (written from a developer’s perspective, not a designer’s), a chapter in the middle of the book will look more directly at interface design and user experience. That material will be anchored by relevant success criteria from the Web Content Accessibility Guidelines (WCAG 2.1) applied to real-time applications. That will help developers keep their applications accessible and touch-friendly from the earliest stages of development. And because WebRTC is also implemented in all modern mobile browsers, the book will spend some time looking at the foundations for building WebRTC applications that are fully responsive and leverage the very best, leading-edge features of HTML, CSS, and JavaScript.

One of the book’s strongest selling points will be its chapter on multi-peer connections. No other topic has as many unanswered or unsatisfactorily answered questions on StackOverflow, Quora, and other venues than how to set up and manage multiple simultaneous peer connections (that is, three or more connected peers). My chapter on the topic will look at establishing multi-peer apps using a mesh-network architecture and only browser APIs. A chapter near the end of book will touch on the theoretical and practical upper limits of multiple simultaneous peer connections. It will guide readers in the `getStats()` method on the `RTCPeerConnection` object for precisely tuning and optimizing bandwidth consumption, especially by multi-peer applications.

WebRTC can easily be misunderstood as a niche subject. That is likely because the most common examples all involve audio and video streaming. My book, however, will include several applications that do not stream audio and video at all, but instead leverage only the `RTCDataChannel` API to transfer data, peer to peer, for file sharing, collaborative work, and even a simple two-player game. There are many novel applications of WebRTC waiting to be discovered. I believe it remains a niche subject only because no book currently exists that is targeted at web developers. My book will provide solid, well researched guidance on WebRTC, written from a web-developer perspective and with a developer mindset. I want my book to be the one to open the floodgates: WebRTC is a game changer, and I am excited to do my part to help get it into the toolbox of workaday web developers, who will do amazing things with it.

To that end, I intend to provide readers with both starter and completed code, a feature I have deeply appreciated in other Pragmatic titles. All of the source code is of my own making and I would hope to liberally license it under MIT or a similar license, so that readers can legally reuse and repurpose the code for their own projects. I also plan to use JSDoc to generate supplemental, web-available documentation that wouldn’t be appropriate for the book, but that might interest some readers.

## Page count (or word count)
I estimate delivering a manuscript of about 50,000 words, excluding inline source code examples, but as with all aspects of this proposal, I am eager to work with the Pragmatic Programmers editorial team to settle on an appropriate length.

## Outline

### Chapter One: The Real-Time Web
This chapter looks at the progression of the real-time web from server-driven (long-polling, Web Sockets) to client-driven, peer-to-peer (WebRTC) technologies. It concludes with some basic setup and a preview of the book’s organization and supporting resources.

### Chapter Two: Establishing a Peer-to-Peer Connection
This whirlwind of a chapter goes a bit light on explanation, which comes elsewhere in the book, in order to focus on establishing an RTC peer connection, covering perfect negotiation and essential code and concepts. Readers end the chapter having built a working peer-to-peer app that streams audio and video.

### Chapter Three: Transmitting Data, Peer to Peer
Beyond streaming audio and video, which is exciting enough, WebRTC supports RTC Data Channels that can be used to transmit arbitrary data, peer to peer. Several different applications, from simple text-based chats to secure file transfer, will illustrate ranges of possible use.

### Chapter Four: Peer-to-Peer Interfaces and User Experience
Having established an RTC Peer Connection and worked with data channels in the previous chapter, readers will be prepared to explore interface patterns and user experiences unique to WebRTC applications, including applications that only make use of RTC Data Channels.

### Chapter Five: Multi-peer Connections
This chapter looks at establishing multiple peer connections on a single call, using a mesh-network architecture, and how to handle peers joining and leaving the call. (This chapter alone will be a huge selling point for the book: the topic is not covered well anywhere—and StackOverflow is full of unanswered posts from people asking how to do exactly what this chapter will cover.)

### Chapter Six: Improved Media Streaming
This chapter takes a closer look at media streams, examining the devices and sources a user can stream from (including the desktop for screen sharing or presentations) and examining the data returned by `RTCPeerConnection.getStats()` to optimize the bandwidth consumed by WebRTC calls.

### Chapter Seven: WebRTC Apps in Production
Closing out the book is a chapter on deploying WebRTC, including setting up and configuring a STUN server and optionally a TURN server, for relaying data when a direct peer-to-peer connection is made impossible by restrictive network firewalls.

## Competing books
Competition for this book is thinner than it appears. The leading title from the most reputable publisher is O’Reilly Media’s book *Real-Time Communication with WebRTC*. But that’s from 2014, and no title I have found from any press of even marginal repute is newer than 2015 (and all are roundly panned on Amazon for having wildly out-of-date code and broken examples, incomplete coverage of WebRTC, etc.). I can’t think of a stronger condemnation of the existing book market than the fact that I wouldn’t in good conscience assign any of these titles to my students: I was pretty desperate for a book, and I created a whole lot of extra work for myself to make up for one.

The WebRTC spec is—as of January 26, 2021—now a W3C Recommendation, after having been at Candidate Recommendation status since November 2017. All of the existing books were written and published years before. All were therefore written according to much earlier working drafts of WebRTC, which has changed significantly on its way to becoming a Candidate and now full W3C Recommendation. Nothing I have seen in the market indicates that new editions are imminent for any of the titles I list below, but publishers aren’t always eager to announce new editions too far in advance, obviously.

For reference, here are the prominent books that developers would probably encounter on their way to discovering and hopefully purchasing my proposed title from Pragmatic Programmers:

* Jonston and Burnett (2014), *WebRTC: APIs and RTCWEB Protocols of the HTML5 Real-Time Web*, 350pp. Digital Codex LLC.
* Ristic (2015), *Learning WebRTC*, 186pp. Packt Publishing.
* Manson (2013), *Getting Started with WebRTC*, 114pp. Packt Publishing.
* Segriienko (2014), *WebRTC Blueprints*, 176pp. Packt Publishing.
* Sergiienko (2015), *WebRTC Cookbook*, 230pp. Packt Publishing.
* Loreto and Romano (2014), *Real-Time Communication with WebRTC: Peer-to-Peer in the Browser*, 164pp. O’Reilly Media.


## PragProg books
My book will complement a number of titles in Pragmatic’s catalog, even as I plan to write it as a standalone book.

With the JavaScript emphasis in the book, and because my own professional practice and how I write about JavaScript has been greatly influenced by them, readers may want to first read these titles:

* Joe Morgan, *Simplifying JavaScript*.
* Venkat Subramaniam, *Rediscovering JavaScript*.
* Trevor Burnham, *Async JavaScript*.

Because I will be making limited use of both Node and ExpressJS, readers may go on to read these titles, or benefit from having read them already:

* Jonathan Lee Martin, *Functional Design Patterns for Express JS*.
* Jim R. Wilson, *Node.JS 8 the Right Way*

Finally, my book might help prepare readers for Stephen Bussey’s *Real-Time Phoenix*--although beyond the signaling server component, WebRTC apps are largely written client side, in the browser.


## Market size
My sense from talking with developers is that WebRTC is known as something that exists, but few seem to know much about it. I expect the market for a book on the topic is likely to grow substantially, now that the specification complete and browser support for WebRTC is so strong and uniform. But here are some rough metrics I was able to assemble:

* GitHub includes almost 19,000 repositories that make some mention of WebRTC.
* The r/WebRTC subreddit has about 2000 subscribers, although WebRTC is routinely mentioned in r/webdev and other web design and deveopment subreddits.
* DevTo’s traffic on the `#WebRTC` tag is modest; metrics are not readily available, but there has been activity at https://dev.to/t/webrtc.
* StackOverflow reports over 500 results (their maximum) for active questions about WebRTC.
* Although there is no one conference that stands out for WebRTC topics, SlideShare reports over 6000 slide decks that mention WebRTC.

The activity on r/WebRTC and StackOverflow is probably most significant: posts in both places frequently feature requests from people just looking for help getting started, or who are looking for a complete, up-to-date guide to go deeper on what they’ve figured out on their own. That represents a real appetite for exactly the kind of book I am proposing.

There’s a real chance that Pragmatic Programmers could be first to market with a book of reasonable length that covers WebRTC according to the final W3C Recommendation. (I’m crossing my fingers that no one is working on such a book for you already, but I can’t wait to read it if they are.) I would do all I can on my end to hand off material to your editorial team whenever they are ready for it. My daily routine includes time for writing and research, which means I’m in the enviable position of not having to squeeze this project in on nights and weekends. As I currently envision the book, I expect to have complete drafts of all chapters by the end of August 2021, and possibly sooner.

## Promotional ideas
I have an extensive and growing professional network of peers as well as a large network of current and alumni students. My university and the professional organizations I belong to (IEEE, ACM) are pretty good about promoting the publications and achievements of their members. I am also active in local developer groups and meetups. I’m not “famous” by any stretch of the imagination, but I’m not a complete unknown, either.

I have a couple of conference talks in the works on WebRTC, and I’m active in web-developer meetup groups in Chicago where I would shamelessly promote the book. For my previous books, I often bought copies at my own expense to hand out at conferences and other events. Hand-to-hand marketing isn't the most efficient, but it can be effective.

That said, I would be eager to learn more marketing strategies from Pragmatic Programmers, and I promise my full cooperation with cold-calling media contacts and outlets, participating in podcast interviews, or even just putting on a sandwich board printed with the book’s cover and standing out on the corner of State and Madison in downtown Chicago.

## Writing sample
I’m including a full chapter, Establishing a Peer-to-Peer Connection, as my writing sample. As the chapter introduction states, its purpose is to get readers up and running with WebRTC as quickly as possible by building a basic video-chat app. The chapter’s many topics will be explored more fully elsewhere in the book, always through additional, lightly featured WebRTC  applications. This chapter both exemplifies how the rest of the book will be written, and showcases how I prepare readers for everything else that follows.
