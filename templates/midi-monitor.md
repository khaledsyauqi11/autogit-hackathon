---
title: MIDI Monitor
app_type: midi-monitor
wallet: 0x0C747c0D04c0379Ff6212A1e164d08B97624f64B
---

Plug in a midi controller. The page lists every connected input, shows every incoming message in real time, draws the active notes across a small piano strip, and writes the most recent thousand messages to a scrolling log. A small synth across the bottom plays the notes through the speakers if the visitor wants to hear what they are pressing, otherwise the page is silent. If no controller is plugged in or the browser does not expose Web MIDI, the page still works, the visitor can tap the piano strip with the mouse and the page treats the taps as a virtual input.

- intro

A hardware black page, the kind of black you find on the back of a synthesizer when the lights are out in the studio. Three accents only. LED amber for the active notes on the piano strip and the small connected indicator on each port. Scope green for the live messages flowing through the monitor list. Port red for clipped or rejected messages. DM Mono at 14px for body, at 11px for the small port labels, and at 22px for the single header line at the top reading midi monitor. Mono only, no other typeface. Weights 400 and 500. Every glyph including the small port plug in the corner and the chevrons in the log filters are inline svg. The page is one column at most 880px wide, centered, 16px gutter on phones.

- inputs

The top of the page below the header is a horizontal strip of input ports. Each port is a card 220px wide and 56px tall, in hardware black with a 1px scope green hairline. Inside the card, two lines. The first line is the port name in 13px DM Mono 500 LED amber, truncated to fit. The second line is the manufacturer in 11px DM Mono muted scope green. A tiny round connected indicator sits at the top right corner of the card in LED amber if the port is open and reading messages, dim if it is closed. Tapping a card toggles whether messages from that port are shown in the log below. By default every port is on the moment it appears.

If the browser exposes the Web MIDI API, the page calls navigator.requestMIDIAccess on a single user gesture (a small text button in the strip reading enable midi inputs, in 12px DM Mono LED amber). The visitor's consent is asked once, the page enumerates every input, listens for connection and disconnection events, and updates the strip live. If the api is unavailable, the strip shows a single port labelled virtual keyboard, manufacturer in this browser only, and the visitor uses the piano strip below as an input.

- the piano

Below the inputs sits the piano strip. Three octaves, C3 to C6, drawn entirely in inline svg. White keys are 28px wide and 120px tall in a pale off black that contrasts with the hardware black page only enough to be readable, with a 1px scope green hairline divider. Black keys are 18px wide and 76px tall, drawn over the white keys at the correct offsets. The default state is dark, no notes lit. When a note on message arrives from any active input, the matching key fills in LED amber with an opacity proportional to the velocity (a velocity of one twenty seven is full opacity, lower velocities are dimmer). A note off, or a note on with velocity zero, removes the fill. The fill clears with a soft 80ms ease in to feel real even at high tempo. Hovering or clicking a key on the piano strip emits a virtual note on, releasing the mouse emits the matching note off. Touch works the same way. Pressing the lowercase letter keys a through k on a qwerty keyboard plays a small white key chord across the bottom octave (a is C4, w is C sharp 4, s is D4, e is D sharp 4, d is E4, f is F4, t is F sharp 4, g is G4, y is G sharp 4, h is A4, u is A sharp 4, j is B4, k is C5), letting the visitor use the page without a controller and without a mouse.

- the monitor

Below the piano strip a single scope green hairline at thirty percent opacity divides the page from the live monitor. The monitor has two regions, the live row and the log.

The live row is a single line, 14px DM Mono LED amber, reading the most recent message in human form like ch 1 note on E4 velocity 96 from akai mpk mini. The line updates on every message, the previous line never lingers, the row is a single live pulse. To the right of the line a small text in 11px DM Mono muted scope green reads the time in seconds and milliseconds since the page loaded, like 4.213 s.

The log is a scrolling list of the most recent one thousand messages, oldest at the top, newest at the bottom, auto scrolled to the bottom by default. Each row is a single line in 12px DM Mono with five columns. The time stamp in muted scope green. The channel number in scope green. The message kind (note on, note off, control change, pitch bend, program change, aftertouch, system) in LED amber for note on, scope green for control change and bend, port red for any message the parser does not recognize. The data bytes in muted scope green, formatted by message kind (note plus velocity for note on, controller plus value for control change). The source port name in italic muted scope green at the end of the row. Tapping a row pauses auto scroll and highlights the row with a thin 1px LED amber outline. A small text button in the top right corner of the log reads resume the live scroll, which returns to the bottom.

Above the log, three small text buttons in 11px DM Mono provide filters. show note messages, show control changes, show everything else. Each button toggles the visibility of the matching kind in the log. The filters persist across the session.

The shape of a parsed message looks like this, written into src/lib/parse.ts.

```
ch 1 note on  E4 vel 96     status 0x90 data 0x40 0x60
ch 1 note off E4 vel 0      status 0x80 data 0x40 0x00
ch 1 cc       7  vol  127   status 0xB0 data 0x07 0x7F
ch 1 bend     +0.18 semi    status 0xE0 data 0x00 0x4A
```

Each row of the log holds the raw bytes, the parsed fields, and the port id of the source.

- the synth

A small text button below the log in 12px DM Mono LED amber reads listen to the notes. Pressing it lazy creates a single WebAudio AudioContext (one per session, gated by user gesture as required by browsers), and routes every active note on through a tiny built in synth. The synth is a single sine plus saw oscillator per note, with an envelope of attack 4ms, decay 80ms, sustain 0.5, release 200ms, and a one pole lowpass at 4 kHz. Polyphony is bounded at sixteen voices, oldest released first when the limit is reached. The output is mixed at twenty percent gain to avoid surprising the visitor. The button toggles between listen to the notes and silence the synth.

A small example sequence that the synth handles cleanly, used as a tiny sanity check in src/lib/sanity.ts at boot.

```
0 ms  ch 1 note on  C4  vel 100
250   ch 1 note on  E4  vel 90
500   ch 1 note on  G4  vel 110
1500  ch 1 note off C4
1600  ch 1 note off E4
1700  ch 1 note off G4
```

- persistence and quirks

The visitor's port filter choices, the active filter buttons (note, cc, other), and the synth on or off state are saved to localStorage under the key midi.monitor.prefs.v1. The log itself is not persisted, every page load starts with an empty log. There is no recording feature, no export, no replay. If localStorage is unavailable, the prefs sit in memory and a tiny italic 11px line at the bottom of the page reads these prefs are not being kept on this device.

Web MIDI messages that arrive from a port the visitor has switched off are not parsed at all, dropped at the listener boundary, to keep the log fast under heavy controllers. The page handles SysEx messages only by displaying their length in bytes in the log, the bytes themselves are not shown, the page does not request the sysex permission. Pitch bend messages are shown as a signed semitone offset assuming a default range of two semitones (the page does not query the controller for an rpn). Program change messages show the program number raw, no general midi names are looked up, the page does not pretend to know what a controller's patches do.

- the build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding hardware, LED amber, scope green, scope green muted, port red. Vite as the build tool. State is plain useState and one useReducer that holds the port list, the active notes set, the log array, and the prefs. No router, no global store, no context provider, no midi library, no synth library, no icon pack. Web MIDI is used directly. WebAudio is used directly. Every glyph including the small plug in the corner and the small play icon on the listen button is inline svg in the component that uses it.

Files. index.html with the DM Mono link in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the midi access, the port list, the live row, the log, and the synth. src/components/Header.tsx for the title and the small plug. src/components/Inputs.tsx for the horizontal port strip. src/components/Piano.tsx for the inline svg piano. src/components/LiveRow.tsx. src/components/Log.tsx for the scrolling list and the filter buttons. src/components/SynthToggle.tsx for the listen button. src/lib/midi.ts for the requestMIDIAccess wrapper, the connect and disconnect handlers, and the listener attachment. src/lib/parse.ts for the message parser. src/lib/synth.ts for the AudioContext, the voice pool, and the envelopes. src/lib/storage.ts for the localStorage try catch. src/lib/sanity.ts for the tiny boot self test sequence. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in chrome on a desktop with no controller attached. The strip is empty except for the enable midi inputs button. They press it, allow access, the strip lights up with their attached akai mpk mini. They press a key on the controller, the piano lights C4, the live row reads ch 1 note on C4 velocity 84 from akai mpk mini, the log gains a row. They press listen to the notes, the small synth comes alive, the next chord they play sings through their speakers at low volume. They unplug the controller, the port card dims, the page keeps working with the virtual keyboard. They reload the tab, the synth toggle and the filters are remembered.
