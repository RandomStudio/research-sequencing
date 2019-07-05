## TITLE
Serious about state / C++ and JS: best friends forever / Native vs Web: why choose? / Solid State (Management)

## BLURB
How we tightened up our state management and sequencing for "video booth" installations
Letting C++ apps do what they're good for, using web tech for the rest
Some lessons we've learned making custom "photo" booth experiences behave reliably

## BODY TEXT

There are quite a number of "moving parts" to a typical video booth installation. Functionality we have to cover:
-  Live video capture (usually in timed bursts to coincide with live experience)
- Editing the video, applying effects and overlays, then encoding into a suitable format
- Creating thumbnails, attaching the video(s), emailing (we like to use proper email infrastructure like [Mandrill](https://mandrill.com/))
- If necessary, also uploading the asset(s) to storage, e.g. [Amazon S3](https://aws.amazon.com/s3/)
- The "experience" itself may include video playback (sometimes synchronised multi-screen content), light control, sensor input, motor control, audio (often multi-channel), text prompts, an iPad for registration (and feedback, e.g. "experience in use") and more...

Very often this can't all run on a single machine, or in any case uses different applications (and programming languages) which are best suited to specific interfaces or processing. So you end up with a kind of [Distributed System](https://en.wikipedia.org/wiki/Distributed_computing) which brings its own special challenges.

