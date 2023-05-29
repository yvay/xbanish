DISCLAIMER: I have no idea what I'm doing. I'm not a C coder. Don't run this
unless you're one and think the changes are acceptable.

This ugly hack lets me run a shell command when the input device changes.

`xbanish -x <device_id> -c "<command>"`

I run it like this:

```
xbanish -t 1 \
-x 10 -c "gsettings set org.gnome.desktop.interface cursor-theme Dot-Dark &" \
-x 17 -c "gsettings set org.gnome.desktop.interface cursor-theme Dot-Dark &" \
-x  0 -c "gsettings set org.gnome.desktop.interface cursor-theme Hackneyed &"
```

This changes the GNOME cursor to a circle when I'm using the stylus (device ids
`10` and `17` according to `xinput list`) and changes it back to the standard
arrow cursor when I'm not (device id `0` is a catch all for all other devices.)

Using both devices at the same time is a convenient way to check the CPU fan is
not broken.

In addition to the original features of xbanish it conveniently hides the
cursor on finger events. Credit for this and other features belong to the
original authors.

# Original readme

## xbanish

xbanish hides the mouse cursor when you start typing, and shows it again when
the mouse cursor moves or a mouse button is pressed.
This is similar to xterm's `pointerMode` setting, but xbanish works globally in
the X11 session.

unclutter's -keystroke mode is supposed to do this, but it's
[broken](https://bugs.launchpad.net/ubuntu/+source/unclutter/+bug/54148).
I looked into fixing it, but unclutter was abandoned so I wrote xbanish.

The name comes from
[ratpoison's](https://www.nongnu.org/ratpoison/)
"banish" command that sends the cursor to the corner of the screen.

### Implementation

If the XInput extension is supported, xbanish uses it to request input from all
attached keyboards and mice.
If XInput 2.2 is supported, raw mouse movement and button press inputs are
requested which helps detect cursor movement while in certain applications such
as Chromium.

If Xinput is not available, xbanish recurses through the list of windows
starting at the root, and calls `XSelectInput()` on each window to receive
notification of mouse motion, button presses, and key presses.

In response to any available keyboard input events, the cursor is hidden.
On mouse movement or button events, the cursor is shown.

xbanish initially hid the cursor by calling `XGrabPointer()` with a blank
cursor image, similar to unclutter's -grab mode, but this had problematic
interactions with certain X applications.
For example, xlock could not grab the pointer and sometimes didn't lock,
xwininfo wouldn't work at all, Firefox would quickly hide the Awesome Bar
dropdown as soon as a key was pressed, and xterm required two middle-clicks to
paste the clipboard contents.

To avoid these problems and simplify the implementation, xbanish now uses the
modern
[`Xfixes` extension](http://cgit.freedesktop.org/xorg/proto/fixesproto/plain/fixesproto.txt)
to easily hide and show the cursor with `XFixesHideCursor()` and
`XFixesShowCursor()`.
