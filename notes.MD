Possibly use C project itself as angle. What have we done so far? What is our latest iteration?

Functionality we have to cover, typically, on photo booth installations
- Capture (usually in timed bursts to coincide with live experience), edit, encode
- Create thumbnails, attach video, email (Mandrill)
- Sometimes upload, e.g. AWS
- Experience may include (multi-screen) video playback, light control, sensor input, motor control, (multi-channel) audio, text prompts, iPad for registration (and feedback, e.g. "experience in use")

In this case, also audio capture and processing.

Where we have come from
- State and logic in snapshot application (and also spread through other components including Node server(s))
- Light / sensor control incorporated partially into snapshot application (but sometimes parts put into Node server where that was "easier"
- Communication typically done with OSC between Node and Cinder apps
- Output on screen often driven by snapshot application, too (video playback)
- Behaviour difficult to re-configure, especially at runtime

Latest iteration
- Keep snapshot application dumb/"stateless" as possible
- Separate light control, sensor data into separate applications (in this case, Cinder app for lighting control, small Node servers for button input, Electron app for "frontend" text display on screen, iPad with registration frontend) - communication to central "control" server via WS
- Sequence defined in JSON
- Promise-based
- Async
- State diagrams (? example images?)
- TypeScript, Redux... anything else new?

Questions
- Was the timeline necessarily linear? Could it incorporate any kind of branching?
- Was "timeline editor" built? Was it used?
- Images available (e.g. state diagrams or anything else used for design stage?)
- Future improvements?
