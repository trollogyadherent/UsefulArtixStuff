# Newbie Arch/Artix tips and tricks
### This is a collection of misc tips, tricks, and fixes that I decided to document alongside my painful journey into Linux. If someone finds it useful, good. Otherwise it will be a lifesaver if I have to reinstall my system. I got meme'd into using Arch + dwm (sorry, Artix, without "soystemd" of course!) right away and I sometimes regret it and sometimes don't. This file is specific to my own machine: Artix base, OpenRC, dwm, Nvidia, Laptop. Fantastic combo if you are concerned about drivers /s
## 
## 

## Wikis
* Depending on the init system you choose, don't be afraid if the Gentoo wiki pops up to one of your Google searches. Some stuff from there applies better than the Arch wiki, because Artix does not use systemd, and Gentoo uses OpenRC and maybe can use other init systems as well

## !! Drivers !!
* If installed linux-lts, install nvidia lts drivers
* Apparently it might sometimes help to have an empty `etc/X11/xorg.conf` file. And it rather *has* to be empty

## Xrandr
* [Mixed monitors xrandr](https://blog.summercat.com/configuring-mixed-dpi-monitors-with-xrandr.html)
* Gui control for `xrandr`: `arandr`

## Correct time (with summer time change)
* Install `ntp` and `ntp-openrc`. Easily configured with `timeset-gui` (AUR)

## Discord emojis (twemoji) on the system and dmenu:
* [System installation](https://partage.les-miquelots.net/engver/blog/how-to-have-colored-emojis-in-dmenu-st-dwm-and-slstatus-on-archlinux.html)
* Dmenu and other suckless: fonts must resemble this: `static const char *fonts[] = {
	"monospace:size=10", "twemoji"
};`
* There is a part of dmenu's code that has to be removed, the comment specifies it blocks colored glyphs

## Virtualbox:
* Currently (probably wrong by now) "official artix stuff" is broken, the script from the official Virtualbox page works

## Bluetooth devices:
* With pulse, proceed as indicated in the [wiki](https://wiki.archlinux.org/index.php/Bluetooth#PulseAudio), that means install `pulseaudio-bluetooth` and add `load-module module-bluetooth-policy
load-module module-bluetooth-discover` at the end of `/etc/pulse/system.pa` and reboot

## Wiimotes/Dolphin:
* If connecting as xbox360 gamepad: Connect using `blueman`. Add wiimote using moltengamepad, use the flag setting it up as an xbox360 device, because otherwise linux sees a wiimote, it's ir and the nunchuk as separate devices (rumble optional): `sudo moltengamepad --mimic-xpad --rumble`
* If using dolphin: do not use `blueman`, it can actually interfere. Just press buttons 1 and 2 while `dolphin` runs. Be sure to select real wiimotes in settings. Connecting wiimotes when a game is already running will not add them to the game.
* Use [NUS Downloader](https://wiibrew.org/wiki/NUS_Downloader) in virtualbox or Windows machine to get the mii channel and launch the wda file like a game, not via the wii system menu.

## DWM:
* One can use multiple modifier keys: `{ MODKEY|Mod1Mask,             	XK_b , 	   					spawn, 		   {.v = browser } },`
* Graph showing different dwm diffs: https://coggle.it/diagram/X9IiSSM6PTWOM9Wz/t/dwm-patches - [archive image](https://i.imgur.com/ACCrQuJ.png)

## Grub
* ~~Graphical config done hassle-free with `grub-customizer`~~ this will break your grub. Don't.

## Glyphs and dwm:
* Colored emojis don't work out of the box, Luke Smith has a [video](https://videos.lukesmith.xyz/w/eCqUSha1rbe56vPgQwqWa5) on it
* `Nerd Font` glyphs appear super small for me, installing `Awesome Font` and `Material Font` was enough and works

## Fixing home perms:
* `# find /home/user/ -type d -print0 | xargs -0 chmod 0777`
* `# find /home/user/ -type f -print0 | xargs -0 chmod 0777`

## Torsocks not working
* Change the port in `/etc/tor/torsocks.conf` to one that your tor broswer uses (`TorPort 9150` should work)

## Resizing the `/swapfile`
* 1) `# swapoff /swapfile` 2) `# fallocate -l XG /swapfile`, where `X` is the amount of gigs you want to add to your swap file. 3) `# swapon /swapfile`

## SUSpending
* Follow [this article](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate) on the Arch wiki
* Use `loginctl suspend` to suspend
* To suspend when pressing the power button, create the `/etc/elogind/logind.conf.d/logind.conf` file. Copy over the contents of `/etc/elogind/logind.conf` to it and uncomment the `#HandlePowerKey=poweroff` line. Change the line to `HandlePowerKey=suspend`
* If you have an Nvidia card (probably only affects proprietary drivers, dunno), open/create `lib64/elogind/system-sleep/nvidia` with your favorite text editor. Change its contents to:
```bash
#!/bin/sh

case "$1" in
    pre)
        /usr/bin/nvidia-sleep.sh "suspend"
        ;;

    post)
        (/usr/bin/nvidia-sleep.sh "resume";)&
        ;;
esac
```

## X server configuration prevents it to start?
* If you autostart X on all tty's, you can liveboot from a key and repair your config, or, much more convenient, SSH into your machine with your phone. Chances are, you don't know your machine's IP address. If you are not a psycho, check it on your router management website. Then, install [Termux](https://f-droid.org/en/packages/com.termux/) on your phone (I assume you have an Android. Otherwise you're on your own here, probably easier to liveboot at this point). Open the app, and install the `openssh` package: `pkg install openssh`. Then connect to your machine using `ssh username@ipaddr`. Example: `ssh jack@192.168.69.69`. In case you get an other prompt than password, type in `yes`. Then, enter your linux machine user password and vim/nano into your .xinitrc file. Or whatever you suspect to break X.
* You can prevent all this fuss by autostarting X only on tty1. Then, if something breaks, just do `ctrl+shift+f2` and edit problematic files from there. To achieve this behavior, start X like this: (notice, I use pulseaudio but if you use something else, omit the `pulseaudio --start` line)
```bash
if [ "$(tty)" = "/dev/tty1" ]; then
	pulseaudio --start
	exec startx
fi
```

## Add directory to path
* This means every script/binary in this directory will be usable right away in your terminal, without you having to specify its full path. Just add something like this to your `.bash_profile`: `PATH=$PATH:/home/jack/Scripts/`. You can have multiple lines like these, specifying another directory each time

## Intellij not working in dwm
* Add `export _JAVA_AWT_WM_NONREPARENTING=1` to your `.bash_profile` file

## Is it me or is one headphone side louder than the other?
* If you use `pulse`, the easiest way is to install `pavucontrol` (should be in Arch/Artix standard repo), run it, select your device under output and click on the lock symbol in the upper right corner. Adjust both sliders to be at the same level.

## I'm trying to install something with `yay` but it has a conflicting dependency
* You can try removing the dependency from the `PKGBUILD` file. For example `freetube` works with `nodejs` instead of `nodejs-lts-x` (at least on the current version). To do this:
* `yay -G <package_name>` - This creates a folder named after a package. `cd` into it.
* Edit the `PKGBUILD` file (remove the conflicting dependency/ies)
* Run `makepkg -si`

## Issues with KDE key-ring (for example error while logging in to github-desktop)
* Install gnome key-ring
* In `.xinitrc`, change `exec dwm` to `exec dbus-run-session dwm`

## Automounting disks
* Check out udisks and `udiskie` (aur). Installing and running it enables mounting drives on plug. Apparently there is an openrc service package but even when not installed it somehow works.
* Udisk does not allow to mount specific drives to specific folders. For that modifying `/etc/fstab` is needed. Find out the uuid of a partition you would like to mount using `sudo blkid`. Get your uid and gid with `id -u` and `id -g`, usually they're both 1000. Then append an entry like this `UUID=01D6C5D8FF828540	/home/jack/Hentai	ntfs	uid=1000,gid=1000,dmask=027,fmask=137` to the `/etc/fstab` file. If you are using another filesystem on that partition, replace ntfs by it. For example ext4.

## Windows 10 Dualboot and Veracrypt overriding boot order
* In Windows, open Veracrypt app, go to settings, advanced options. Uncheck `Force VeraCrypt entry to be the first in the EFI firmware boot menu` and `Automatically fix boot configuration issues that may prevent Windows from starting`. Now boot into Linux and use `efibootmgr` to set the VeraCrypt and Windows Boot Manager as inactive. Grub should now always appear at boot.

## Lutris, esync thing
* Follow this video: https://yewtu.be/watch?v=AVqsdO7xENg / [post](https://forum.artixlinux.org/index.php/topic,3336.0.html)/[archive](https://archive.ph/jJPyt) , it is based on these sources: https://wiki.artixlinux.org/Main/Repositories [archive](https://archive.ph/hpjoV), https://forums.gentoo.org/viewtopic-t-1111528-highlight-ulimit.html [archive](https://archive.ph/yZ4M5), https://www.christitus.com/ultimate-linux-gaming-guide/ [archive](https://archive.ph/cpMyn), https://github.com/AUNaseef/protonup

## Can't open X apps after a while / .Xauthority breaks
* From the Gentoo Wiki: When NetworkManager connects to a WiFi access point, it might change your hostname. If it does, it might mess with your X authentication and prevent you from launching X applications. You can verify this with xauth list.
*To fix this, you can set `hostname-mode = none` in your config (`/etc/NetworkManager/NetworkManager.conf`).

## Mapping controllers to xbox360
* Bought some shitty plastic controller but of course the mappings are nonsense? Install `gamepad-tool-bin` from the AUR and launch it.
* Plug in controller, click "Generate a new mapping". When you are done, click on "Copy Mapping String". Save it in some file, her it is `~/mappings/my_mapping`
* In your `.bash_profile` append the following line: `export SDL_GAMECONTROLLERCONFIG=$SDL_GAMECONTROLLERCONFIG:`cat ~/mappings/my_mapping``. This adds the mapping to the SDL environment variable. Yup, it is indeed stored in a long-ass env var. If you want to map other stuff, just append the same export line, just change the mapping file.

## pip search
* `pip search` was discontinued. Use [pip-search](https://pypi.org/project/pip-search/). Add the snippet provided on the website to `.bashrc` and you'll be able to use `pip search <query>` again

## Use playerctl with mpv
* Download `mpris.so` from [here](https://github.com/hoyon/mpv-mpris/releases)
* Place it in `~/.config/mpv/scripts`

# Other/Misc

## Handy programs:
* Calculator: `qalculate` (cli), `qalculate-gtk` (gui)
* Browser: `firefox`, `brave`, `chromium`, `lynx` (cli, y tho)
* Image manipulation: `gimp`, `pinta` (sucks tbh)
* Office: `libreoffice`
* Mail: `thunderbird`, `claws-mail`, `neomutt` (cli)
* Media consoomption: `mpv`, `vlc`
* PDF viewing: `zathura`
* Image viewing: `sxiv`
* Text editing: `sublime`, `neovim`


## Ricing Guides:
* [dwm status](https://www.youtube.com/watch?v=6vTrVPpNodI)
* [urxvt](https://www.youtube.com/watch?v=OVko_lhkQjs)

## Extra Resources
* [Synopta](https://addons.mozilla.org/en-US/firefox/addon/add-custom-search-engine/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search)
* [https://wiki.installgentoo.com/wiki/List_of_recommended_GNU/Linux_software](https://wiki.installgentoo.com/wiki/List_of_recommended_GNU/Linux_software)
* [https://suckless.org/sucks/](https://suckless.org/sucks/)
* [https://suckless.org/rocks/](https://suckless.org/rocks/)
* [https://privacytools.io/](https://privacytools.io/)
* [https://www.fsf.org/resources/](https://www.fsf.org/resources/)
* [https://www.gnu.org/software/software.html](https://www.gnu.org/software/software.html)
* [https://github.com/mayfrost/guides/blob/master/ALTERNATIVES.md](https://github.com/mayfrost/guides/blob/master/ALTERNATIVES.md )

## Misc
* [some urxvt dotfiles](https://github.com/charnley/dotfiles.x/blob/master/Xresources)
* [Smoking Spectre dotfiles](https://github.com/SmokinSpectre/Dots)
* [wolfiy dotfiles](https://gitlab.com/wolfiy)

## Notes/TODO:
* Rice dwm and dstatus, ~~patch dwm so the status is on all screens~~
* setup and rice [these](https://github.com/dudik/herbe) toasts. They can be killed with a precise `kill` command, bind it to dwm and voil??
* daemonize isync


