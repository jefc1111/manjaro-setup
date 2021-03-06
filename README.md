# manjaro-setup
## Notes from setting up Manjaro a few times on a few different Thinkpads

### Initial setup 
- Boot to USB stick, enter wifi details and select install from the modal
- Reboot into new Manjaro install and connect to wifi
- Click the thing in the tray to bring up software manager. Install all updates. 
- Click the three dots menu and allow install from AUR
- Run `sudo pacman -Syy` to update package databases


### App installs
- autorandr
- flameshot
- peek
- VS Code (visual-studio-code-bin)
- Remove palemoon
- Brave
- Slack (slack-desktop)
- Add Bitwarden to Brave and pin it to the thingy bar
- Studio 3t
- MySQL Workbench
- Mongo server (see https://wiki.archlinux.org/index.php/MongoDB)

### Tweak touchpad responsiveness
- `xinput` to list devices
- `xinput list-props <id of device>` using id for your touchpad
- `xinput set-prop <id of device> <id of prop called libinput Accel Speed> 0.5` 0.5 probably too sensitive. Depending on the touchpad I have used values between about 0.3 and 0.8 (the latter on the W500). This will change it straight away, so nice and easy to tweak. It's sad, but you are going to lose this setting on restart so...

Make touchpad setting permanent
- `sudo nano /etc/X11/xorg.conf.d/30-touchpad.conf` and add `Option "AccelSpeed" "0.8"`

### Making the touchscreen work (X1 Extreme gen 3)
Without doing this, I found on dual screens, the touch input thought it had the run of both screens, so would be offset (unless at the very botoom where it would 'catch up with itself')
- `xinput` and find id of touchscreen
- `xinput map-to-output <relevant_output_id> eDP1` changing parameters as required

Trouble is, the relevant id in `xinput` is not stable, so we have to get the id dynamically with some `awk` trickery;
```xinput map-to-output `xinput | awk '/multitouch sensor Finger/ {print $9}' | grep -o '[0-9]\+'` eDP1```

This should be set to run on every dock / undock and on wake from suspend.

### xrandr / autorandr set up
This was complex for me because my built in screen is 4k (3840 × 2160) while the external screen is 2560x1440
While docked, I am ok with the built in screen running at 1920x1080, but when undocked I want it at 3840 × 2160, but suitably scaled. 
See the config file folder for more details.
I also set `autorandr-c` to run on resume by following this https://archived.forum.manjaro.org/t/solved-run-command-upon-resume/58588/5

### Annoying blip in headphones in URxvt
Add `xset b off` to `~/.bashrc`

### Showing slightly wrong time
`sudo pacman -S ntp`  
`sudo timedatectl set-ntp true`

### Miscellaneous
Add `alias ll="ls -al"` to `.bashrc` (don't forget to `source` it!)  

In `.profile` change default browser to `brave`  

### i3wm config  
To start some apps I have this right at the bottom of the file;
```
exec --no-startup-id flameshot
exec --no-startup-id urxvt
exec --no-startup-id brave
exec --no-startup-id code
exec --no-startup-id slack
exec "sleep 10; /home/jef/my-scripts/after-startup.sh"
```

The contents of `after-startup.sh` is;
```
#!/bin/bash
i3-msg [class="Brave-browser"] move container to workspace 2
i3-msg [class="Slack"] move container to workspace 3
i3-msg [class="Code"] move container to workspace 4
```

Aside: Various other solutions for pushing specific apps to different workspaces meant that _all_ windows for the given application would always open on the associated workspace. I want to be able to open a terminal next to my browser and things like that. 

Disable screen locking (look for `xautolock` and comment it out)  

Disable nitrogen

Change conky to the green version (should be a commented out line that can be swapped with currently enabled one)

### Disable screen lock when suspending
Took ages to work this one out. There is a script called `i3exit` in `/usr/bin` that is referred to in the the i3 config file. It's in this script that `blurlock` is invoked. So just go ahead and edit that file, removing the call to `blurlock`

### Getting autologin working (W500)
Even though I selected autologin at OS setup it still wasn't working. I went round the houses trying looooads of stuff. In the end I have this;

In /etc/lightdm/lightdm.conf
```
pam-service=lightdm-autologin
#pam-autologin-service=lightdm-autologin
#pam-greeter-service=lightdm-greeter
autologin-guest=false
autologin-user=jef
autologin-user-timeout=0
```

In /etc/pam.d/lightdm I added
```
auth        sufficient  pam_succeed_if.so user ingroup nopasswdlogin
auth        include     system-login
```
I also made sure my user was in groups `autologin` and `nopasswdlogin`. 
I doubt _all_ of this was necessary. I got stuck for a while where I would still get a login screen but was able to click <enter>. The fix for this was commenting out `pam-greeter-service` in lightdm config (as above). This was a step which I hasn't specifically seen recommended anywhere. 

### Virtualization (X1 Extreme gen 3)
There was no kernel module for Virtualbox for the very recent kernel I am using, so I switched to `libvirt` / `kvm`.

Then use `vagrant up --provider=libvirt`

To avoid always being asked for root password I changed permissions on `/etc/exports` and also followed this: https://computingforgeeks.com/use-virt-manager-as-non-root-user/

### Taking control of both fans (X1 Extreme gen 3)
Had to switch to Manjaro testing branch and then upgrade to 510 kernel: https://www.reddit.com/r/thinkpad/comments/kadzkb/an_optimistic_x1_extreme_fan_noise_post_linux/

Thinkfan: 
Had to tweak `/etc/systemd/system/thinkfan.service.d` to `Environment='THINKFAN_ARGS=-b-3'` to prevent annoyingly frequent changes to fan speed. 

`/etc/thinkfan.conf`;
```ensors:
  - hwmon: /sys/devices/platform/coretemp.0/hwmon
    indices: [1, 2, 3, 4, 5]

fans:
  - tpacpi: /proc/acpi/ibm/fan

levels:
  - [0, 0, 40]
  - [1, 40, 50]
  - [2, 50, 55]
  - [3, 55, 60]
  - [4, 60, 65]
  - [5, 65, 70]
  - [7, 70, 32767]
```

### Unable to adjust brightness and no backlight after suspend (P51s)
Lots of potential solutions floating around. For me it was a case of doing this https://forum.manjaro.org/t/cant-adjust-screen-brightness-lenovo-laptop-nvidia-xfce/28771/13


#### Notes
Use `xprop` to find the class name for moving apps to workspaces and so on (https://www.reddit.com/r/i3wm/comments/3h94t9/how_to_find_a_name_of_a_program_to_use_for/)






