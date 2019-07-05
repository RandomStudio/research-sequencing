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

Our most recent iteration focussed on two main areas of improvement:
1. **Break up the system into smaller, focussed pieces.** Let applications be focussed on a single area of responsibility, as far as possible.
1. ** 