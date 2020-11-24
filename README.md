# manjaro-setup
## Rough aide-memoire for how I set up Manjaro for me

Boot to USB stick, enter wifi details and select install from the modal

Reboot into new Manjaro install and connect to wifi

Click the thing in the tray to bring up software manager. Install all updates. 

Click the three dots menu and allow install from AUR

Install Flameshot

Install Peek

Install VS Code (visual-studio-code-bin)

Remove palemoon

Install Brave

Install Slack (slack-desktop)

Now tweak touchpad responsiveness
- `xinput` to list devices
- `xinput list-props <id of device>` using id for your touchpad
- `xinput set-prop <id of device> <id of prop called libinput Accel Speed> 0.5` 0.5 probably too sensitive. Depending on the touchpad I have used values between about 0.3 and 0.8 (the latter on the W500). This will change it straight away, so nice and easy to tweak. It's sad, but you are going to lose this setting on restart so...

Make touchpad setting permanent
- `sudo nano /etc/X11/xorg.conf.d/30-touchpad.conf` and add `Option "AccelSpeed" "0.8"`

Add `alias ll="ls -al"` to `.bashrc` (don't forget to `source` it!)

In `.profile` change default browser to `brave`

Now to edit i3wm config
TODO: Add changes that start the apps 

Add Bitwarden to Brave and pin it to the thingy bar


TODO: Prevent need login
TODO: Prevent screensasver






