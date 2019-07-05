## TITLE
Solid State (Management)

## BLURB
Some lessons we've learned making custom "photo" booth experiences behave reliably

## BODY TEXT
### Simple concept, complex architecture
One of the types of installations we've made fairly often at Random Studio is a video ("photo") booth. For example, take a look at [A vertical journey through time for Chanel](https://www.random.studio/projects/chanel-timebox).

The concept is quite straightforward: let somebody begin an "experience", capture some video footage, and send them an edited video (a "digital takeaway") attached to a nicely-formatted email.

A lot needs to happen, in terms of hardware and software, in order to make this work. And since our clients are typically looking for a bespoke experience, there is a large amount of custom development involved.

There are quite a number of "moving parts" to a typical video booth installation. Functionality we have to cover:
- **Integrating high quality cameras and video capture technology**. We use a lot of [BlackMagic Micro Cinema](https://www.blackmagicdesign.com/nl/products/blackmagicmicrocinemacamera) cameras and the [BlackMagic SDK](https://www.blackmagicdesign.com/nl/developer/)
-  **Live video capture** (usually in timed bursts to coincide with live experience). For this we use an in-house framework called "Snapshot" which is developed in C++ in the [Cinder](https://libcinder.org/) creative coding framework.
- **Editing the video, applying effects and overlays, then encoding** into a suitable format. We use [ffmpeg](https://ffmpeg.org/), kicked off as a process by a [NodeJS](https://nodejs.org/en/) server.
- Creating thumbnails, attaching the video(s), **emailing** (we like to use proper email infrastructure like [Mandrill](https://mandrill.com/), and we use their NodeJS API).
- If necessary, also **uploading** the asset(s) to storage, e.g. [Amazon S3](https://aws.amazon.com/s3/)
- The "experience" itself may include all kinds of **hardware intefacing**, e.g. for video playback (sometimes synchronised multi-screen content), light control, sensor input, motor control, audio (often multi-channel), text prompts, an iPad for registration (and feedback, e.g. "experience in use") and more...

Very often this can't all run on a single machine, or in any case uses different applications (and programming languages) which are best suited to specific interfaces or processing. So you end up with a kind of [Distributed System](https://en.wikipedia.org/wiki/Distributed_computing) which brings its own special challenges.

### How we did things before
Although previous installations have ultimately performed well, they could be extremely **difficult to troubleshoot and debug** in some scenarios. 

It was also typically **difficult to make content, design and timing changes** without involving a developer. Often, even minor changes required re-compiling code.

These difficulties had various, somewhat related causes:
- State and logic resided largely in the "Snapshot" (C++, Cinder) application, but these concerns were also handled in some other components including Node servers written in JavaScript
- Light control (via ArtNet/DMX) and sensor control (via serial/USB) was sometimes incorporated into the Snapshot application itself, but sometimes these were put into the same monolithic Node server where that seemed "easier" to implement
- Communication was typically done with OSC between Node and Cinder apps, because OSC communication is easy to do in Cinder but Websocket communication required third party libraries
- Output on screen (video playback) was often often driven by the snapshot application, too 

The common themes above: a fuzzy, decentralised approach to state management, and a confusing mix of commmunication protocols being used between different parts of the system.

The behaviour and sequencing of the experience and "digital takeaway" generation were difficult to re-configure at runtime. This was especially true of the Cinder/C++ application, where JSON parsing is more cumbersome and configuration utilities often need to be built from scratch.

## The shift: split up and re-centralise
In contrast with most C++ development, web application development offers a mature set of options for [state management](https://www.quora.com/What-does-state-management-even-mean-and-why-does-it-matter-in-Front-End-Web-Development-with-frameworks-like-React-or-Vue) frameworks. Some solid "best practices" have developed in the industry because of the constant need to deal with complex user interactions and two-way communication (between frontend and backend, client and server, or between services in a [microservice architecture](https://en.wikipedia.org/wiki/Microservices)).

We believed it was time to apply some of these lessons to make our video booths easier to develop and maintain.

Our most recent iteration focussed on four main areas of improvement:
1. **Break up the system into smaller, focussed pieces.** Let applications be focussed on a single area of responsibility, as far as possible.
1. **Standardise communication protocols.** Use [websockets](https://en.wikipedia.org/wiki/WebSocket) for communication in the system, as far as practical, whether between applications on the same machine or over a real network. Websockets also have the advantage of allowing easy two-way communication.
1. **Centralise state management.** Let a single Node Server direct the most important events and mode transitions in the system. Remove state and hardcoded logic (decision-making) from other components as far as possible.
1. **Separate configuration (what should happen) from the engine (how the sequencing, etc. should be applied).** Make it possible for non-coders to change how the system behaves to a large extent, without involving coders and without risking errors and undetermined behaviour.

## How we did it
### Breaking things up
We "dumbed down" our C++/Cinder "Snapshot" application a lot. We kept it as near to "stateless" as we could manage. The Snapshot application would only trigger its audio and video capturing on command from the server, and return messages about status (capture complete) and locations of processed assets, etc.

We pulled functionality such as **light control and sensor data** into separate applications: in this case, another Cinder app for lighting control, small Node servers for the conductive touch "button" input, an Electron app for "frontend" text display on screen and a website running in a browser on an iPad (for user registration). We even had a small server that was responsible only for listening to button events from a "reset" button, to trigger a system reboot. 

### Standardising communication
All of these pieces of software listed above communicated to a central "control" server via websocket connections.

### Centralising state management
Our team implemented a [Redux](https://redux.js.org/) state management container on the control server. This makes the system extremely testable and much easier to debug. The software was hardened further by the use of [TypeScript](https://www.typescriptlang.org/).

### Easy (re)configuration
We put multi-region / multi-language "content" (in this case, on-screen prompts and references to voice-over clips) in a [YAML](https://yaml.org/) file: easy to read and edit, even for non-coders.

The actual "sequencing" behaviour was put into a separate JSON file, which was essentially a list of "tasks" defining the behaviour of the booth. For example, a task might specify that a text prompt should go up on the display (with a reference to the correct section in the content file), and a sound play. Or it could specify that a 5 second audio clip should be recorded. Or a 15 second video clip. Lighting animations and all aspects of timing could be changed without modifying any code in the server (or anywhere else in the system).

To actually generate and modify the JSON file, **we created a web-based editor**, which allowed designers, producers, creatives (non programmers) to build out a sequence, modify tasks and change their order without having to touch the actual data file directly.

The "engine" to run this sequence made use of a newly-standardised (but very mature) feature of JavaScript, [Promises](https://developers.google.com/web/fundamentals/primers/promises). The sequence array would be parsed and then queued up, allowing for different methods of "resolving" each task, e.g. a condition was met (a user pushed the button when prompted) or a timeout completed.

The result was a flexible and reliable sequencer that allowed designers to focus on content and timing, while coders could focus on stability, performance and functionality. And these areas of responsibility could be managed largely independently and in parallel.

## The conclusion
We ended up with a video booth system that was exceptionally stable, easier to test and largely immune to the kinds of bugs we used to see in earlier iterations. 

It was also much easier (and faster, and less risky) to make all kinds of changes to the experience sequence without having to write code.

## What's next
A few improvements we can think of, which we may try to tackle in the future:
- make the sequencing system more flexible, in terms of the types of tasks that can be set up
- a more "visual" (less form-based) layout for the editor
- use [monorepos](https://en.wikipedia.org/wiki/Monorepo) to consolidate the disparate parts of the system for version control and deployment
- apply more strict typing (and more extensive use of [interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html)) for TypeScript, especially on the server side
- allow for more sophisticated non-linear flow, e.g. branching, in the sequencer
