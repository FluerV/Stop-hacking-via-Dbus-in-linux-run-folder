Prevent hacking via Dbus in linux  /run/user folder

In most cases hackers tries to use low-level dbus-daemon (chat or something similar) for sending and receiving theirs messages. In most linux distro you can't complitelly remove dbus-daemon without breaking your desktop environment. Вut you should reduce activity of per-session message bus as much as possible. When you notice third-party activity on your Linux machine you need to look at some important directories and files, which bring per-session message bus to the active session.
Note: this instruction doesn't complitelly remove hacker's activity from your machine but can reduce it. Remember: IT security is a process, not a result!:)
Check active users in "run/user" folder
This directory include user's files, compiling "on-the-fly". You can check users directories in /run/user folder manually or via "systemctl" command (sometimes hackers doesn't put phisicalies folders).  If you are only user of your linux computer this directory should include /run/user/1000 folder and nothing else. 
But if you see another one folder here with UID less then 1000 it's time to start worries (example: /run/user/113).
2. Prevent activity of per-session message bus

2.1 in startx point Dbus to /dev/null 

In /usr/bin/startx file change the line "unset DBUS_SESSION_BUS_ADDRESS" to "DBUS_SESSION_BUS_ADDRESS=/dev/null".

2.2 in etc/xdg/xfce4/xinitrc find and comment (delete) the following lines:

# Use dbus-launch if installed.
#if test x"$DBUS_SESSION_BUS_ADDRESS" = x""; then
#  if which dbus-launch >/dev/null 2>&1; then
#    eval `dbus-launch --sh-syntax --exit-with-session`
    # some older versions of dbus don't export the var properly
#    export DBUS_SESSION_BUS_ADDRESS
#  else
#    echo "Could not find dbus-launch; Xfce will not work properly" >&2
#    fi
#fi


2.3  Remove or edit dbus-x11 and dbus-user-session packages

These packages brings Dbus activity in Xsession's environment. It's better to remove both. You can purge dbus-x11 without problems. 
But removing dbus-user-session will breaks your system (because of hard dependencies) and bring it in single user mode.  
dbus-user-session checks active users in /run/user folder and bring per-session message bus in the desktop environment.
You can kill  dbus-user-session activity by editing config file in /etc/X11/Xsession.d/20dbus_xdg-runtime (see below) or 
 comment all lines.  
 
# vim:set ft=sh sw=2 sts=2 et:

if [ -z "$DBUS_SESSION_BUS_ADDRESS" ]; then 
  # We are under systemd-logind or something remarkably similar, and
  # a user-session socket has already been set up.
  #
  # Be nice to non-libdbus, non-sd-bus implementations by using
  # that as the session bus address in the environment. The check for
  # XDG_RUNTIME_DIR = "/run/user/`id -u`" is because we know that
  # form of the address, from systemd-logind, doesn't need escaping,
  # whereas arbitrary addresses might.
  DBUS_SESSION_BUS_ADDRESS="unix:path=/dev/null"
  export DBUS_SESSION_BUS_ADDRESS
fi

#if [ -x "/usr/bin/dbus-update-activation-environment" ]; then
  # tell dbus-daemon --session (and systemd --user, if running)
  # to put a minimal subset of the Xsession's environment in activated
  # services' environments
 #dbus-update-activation-environment --verbose --systemd \
   #DBUS_SESSION_BUS_ADDRESS DISPLAY XAUTHORITY
#fi  

2.4 File  etc/x11/Xsession.options should looks like that:

# $Id: Xsession.options 189 2005-06-11 00:04:27Z branden $
#
# configuration options for /etc/X11/Xsession
# See Xsession.options(5) for an explanation of the available options.
allow-failsafe
#no-allow-user-resources
#no-allow-user-xsession
#no-use-ssh-agent
#no-use-session-dbus

2.5 in /etc/xdg/autostart search for ..ssh.desktop, ..gnome-keyring and similar files and delete packages. 

2.6 Remove gnome (apt purge gnome*), pinentry-gnome3, libatk-adaptor, at-spi2-core and other packages that brings remote acsess and dbus activity  to your machine.
You can use "apt purge --auto-remove <package>" to delete config and unnecessary files. But be careful with autoremove. 

If sometimes you can't find package's file to edit it or remove package because it broken run this set of command (root):

dpkg --purge --force-depends <package>
apt install <package>

or

apt --fix-broken install

2.7 Don't use gnome, kde etc. The most safe desktop environment is xfce. 

(Link on Geoclue). 

3. Some helpfull command (root).

3.1 To understand what package a file belongs to:

dpkg -S <package name>

3.2 To list all package's file:

dpkg -L <package name>

3.3 To see all info about package:

 apt-cache show <package name>

3.4 Backup your machine with Clonezilla. 

3.5 Sometimes you can break your system, removing some packages. Don't worry. You can restore your Clonezilla backup. If there is no backup you can boot in single user mode and write in terminal:
apt --fix-broken install

For that reason it's much better to install linux from DVD (Flash). If your broken system doesn't connect to the internet you always can use your DVD (Flash) installation disk and repaire your linux machine with "apt --fix-broken install" in command line.
4. Final thoughts

After these steps  unwanted user will dissapier from /run/user folder, but hacker can use extra session (for example session 2 of legal user), temporarily join to your display "on-the-fly" via ssh  etc. 

Wish you keep your machine safe. Good luck!








