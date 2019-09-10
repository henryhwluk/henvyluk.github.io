---

title: Ubuntu VNC Gnome
urlPath: ubuntu-vnc-gnome
date: 2019-06-06
updated: 2019-06-06
tag: [Ubuntu, linux, Gnome, VNC]

---

### SSH Login
* `ssh root@ip`
* `apt upgrade -y`

### Gnome 2
* Complete installation
 `apt install ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal -y`

* Incomplete installation `apt-get install --no-install-recommends ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal -y`

<!-- more -->

### VNC

* `apt install vnc4server -y`
* `vncserver :1`
* `vncserver -kill :1`
* `vim /etc/vnc/xstartup`
* find `x-window-manager & `line add
 `gnome-panel &`
`gnome-settings-daemon &`
`metacity &`
`nautilus &`
* full config

```
#!/bin/sh
 	 
 	# Uncomment the following two lines for normal desktop:
 	# unset SESSION_MANAGER
 	# exec /etc/X11/xinit/xinitrc
 	 
 	[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
 	[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
 	xsetroot -solid grey 
 	vncconfig -iconic &
 	x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
 	x-window-manager &
 	gnome-panel &
 	gnome-settings-daemon &
 	metacity &
 	nautilus &
```
### Mac VNC
* `ip: displayId`

### Too many security failures
* kill vncserver
   * `vncserver -kill :1`
   * `vncserver :1`

* reset blacklist
   * `vncconfig -display :1 -set BlacklistTimeout=0 -set BlacklistThreshold=1000000`
   * `vncconfig -display :1 -set BlacklistTimeout=100000000000 -set BlacklistThreshold=10`


 
