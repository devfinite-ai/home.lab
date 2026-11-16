# Controller design notes

These are hardware-integration lessons from the Home Productivity prototype.
They apply broadly to MIDI devices used as computer control surfaces.

## A button press is not always one event

A control may emit a press, release, aftertouch, or repeated samples while held.
Some toggle controls alternate between values such as `127` and `0`; both can
represent consecutive physical presses rather than press and release.

A reliable bridge therefore needs control-specific edge handling instead of a
single rule for all buttons.

## Debounce is necessary but not sufficient

A short time window suppresses electrical or software chatter, but timing alone
can also discard a legitimate fast second press. Combine it with:

- Expected MIDI message type and channel
- Allowed value transitions
- Same-value echo suppression
- A single pending action per physical gesture

## LEDs should show confirmed state

Hardware toggle lights can drift from the operating system when an action fails,
another application changes state, or the controller maintains its own toggle
memory. Treat Windows as the source of truth:

1. Request the action.
2. Read the resulting Windows state.
3. Update the controller LED from that confirmed state.
4. Periodically resynchronize long-lived controls such as microphone mute.

## Foreground actions need a stable target

Keyboard and scroll input should not be sent merely because an application was
foreground a moment ago. Resolve the foreground window and focused control,
reject unsafe shell or lock-screen surfaces, then verify the foreground has not
changed immediately before delivery.

## Prefer semantic actions over keystroke macros

Use Windows APIs for system state such as audio, media, and window activation.
Reserve keystrokes for application commands that lack an API, and constrain
those commands to a verified foreground application.

## Keep personal configuration out of source

Controller exports and mapping files often contain executable paths, profile
names, URLs, or device-specific identifiers. Public examples should use obvious
placeholders and provide an installation step that fills local values without
modifying the tracked template.
