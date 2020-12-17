# manjaro-setup
## Rough aide-memoire for how I set up Manjaro for me

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

### Tweak touchpad responsiveness
- `xinput` to list devices
- `xinput list-props <id of device>` using id for your touchpad
- `xinput set-prop <id of device> <id of prop called libinput Accel Speed> 0.5` 0.5 probably too sensitive. Depending on the touchpad I have used values between about 0.3 and 0.8 (the latter on the W500). This will change it straight away, so nice and easy to tweak. It's sad, but you are going to lose this setting on restart so...

Make touchpad setting permanent
- `sudo nano /etc/X11/xorg.conf.d/30-touchpad.conf` and add `Option "AccelSpeed" "0.8"`

### Making the touchscreen work (on X1 Extreme gen 3)
Without doing this, I found on dual screens, the touch input thought it had the run of both screens, so would be offset (unless at the very botoom where it would 'catch up with itself')
- `xinput` and find id of touchscreen
- `xinput map-to-output 10 eDP1` changing parameters as required

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



### Getting autologin working (this was on my W500 only)
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

### Virtualization (X1 Extreme)
There was no kernel module for Virtualbox for the very recent kernel I am using, so I switched to `libvirt` / `kvm`.

Then use `vagrant up --provider=libvirt`

To avoid always being asked for root password I changed permissions on `/etc/exports` and also followed this: https://computingforgeeks.com/use-virt-manager-as-non-root-user/






TODO: AWS cmd line tools & my IP script  
TODO:   
install mysql workbench    
install studio 3t    




#### Notes
Use `xprop` to find the class name for moving apps to workspaces and so on (https://www.reddit.com/r/i3wm/comments/3h94t9/how_to_find_a_name_of_a_program_to_use_for/)






