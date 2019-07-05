Possibly use C project itself as angle. What have we done so far? What is our latest iteration?

Where we have come from
- State and logic in snapshot application (and also spread through other components including Node server(s))
- Light / sensor control incorporated partially into snapshot application (but sometimes parts put into Node server where that was "easier"
- Communication typically done with OSC between Node and Cinder apps
- Behaviour difficult to re-configure, especially at runtime

Latest iteration
- Keep snapshot application dumb/"stateless" as possible
- Separate light control, sensor data into separate applications (in this case, Cinder app for lighting control, small Node servers for button input) - communication to central "control" server via WS
- Sequence defined in JSON
- Promise-based
- Async
- State diagrams (? example images?)
