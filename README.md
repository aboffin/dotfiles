# dotfiles
The obligatory dotfiles repo.
XMonad, acme plumb configuration files (OpenBSD/Slackware)
# .xinitrc
#!/bin/sh

userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
sysresources=/etc/X11/xinit/.Xresources
sysmodmap=/etc/X11/xinit/.Xmodmap

# Merge in defaults and keymaps
[ -f $sysresources ] && /usr/bin/xrdb -merge $sysresources
[ -f $sysmodmap ] && /usr/bin/xmodmap $sysmodmap
[ -f $userresources ] && /usr/bin/xrdb -merge $userresources
[ -f $usermodmap ] && /usr/bin/xmodmap $usermodmap
# Change mouse settings to left-handed
xmodmap -e "pointer = 3 2 1"
# Change mouse pointer to right-pointing arrow
xsetroot -cursor_name right_ptr
/opt/plan9/bin/plumber
if [ -z "$DESKTOP_SESSION" -a -x /usr/bin/ck-launch-session ]; then
  exec ck-launch-session dbus-launch --exit-with-session /usr/bin/xmonad
else
	exec /usr/bin/xmonad
fi


# xmonad.hs (Slackware)
import XMonad
import XMonad.Hooks.DynamicLog
import XMonad.Hooks.ManageDocks
import XMonad.Util.Run(spawnPipe)
import System.IO
import XMonad.Config.Azerty
main = do
    xmproc <- spawnPipe "/usr/bin/xmobar /home/senthil/.xmobarrc"
    xmonad azertyConfig { manageHook = manageDocks <+> manageHook defaultConfig, layoutHook = avoidStruts  $  layoutHook defaultConfig, logHook = dynamicLog, modMask = mod4Mask }

# .xmobarrc (Slackware)
Config { font = "fixed"
       , borderColor = "black"
       , border = TopB
       , bgColor = "black"
       , fgColor = "grey"
       , position = Top
       , allDesktops = True
       , lowerOnStart = True
       , pickBroadest = False
       , overrideRedirect = True
       , persistent = True
       , hideOnStart = False
       , commands = [ Run Battery ["-t","AC: <acstatus> Batt: <left> <timeleft>"] 10
                    , Run Com "uname" ["-s","-r"] "" 36000
                    , Run Date "%a %b %_d %Y %H:%M:%S" "date" 10
                    ]
       , sepChar = "%"
       , alignSep = "}{"
       , template = "%battery% }{ <fc=#ee9a00>%date%</fc> | %uname%"
       }

# acme plumbing
# for some reason, it works only the following way 
# i.e., keeping almost the same settings in two files separately

# lib/plumbing.b
# to update: cat $HOME/lib/plumbing.b | 9p write plumb/rules
# and cat $HOME/lib/plumbing | 9p write plumb/rules
editor = acme
include basic

# postscript/pdf/dvi go to page but not over the a plumb port
# the port is here for reference but is unused
type is text
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.(pdf|PDF)'
arg isfile	$0
plumb to mupdf
plumb start mupdf $file

# open office - s[xt][cdigmw], docx?, xlsx?, pptx?
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.([Ss][XxTt][CcDdIiGgMmWw]|[Dd][Oo][Cc][Xx]?|[Xx][Ll][Ss][Xx]?|[Pp][Pp][Tt][Xx]?)'
arg isfile	$0
plumb to openoffice4
plumb start openoffice4 $file

# image files go to mupdf
type is text
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.(jpe?g|JPE?G|gif|GIF|tiff?|TIFF?|ppm|bit|png|PNG)'
arg isfile	$0
plumb to mupdf
plumb start mupdf $file

# relative files as file: urls get made into absolute paths
type is text
data matches 'file:([.a-zA-Z¡-￿0-9_\-]([.a-zA-Z¡-￿0-9_/\-]*[a-zA-Z¡-￿0-9_/\-]))?'
arg isfile	$1
data set	file://$file
plumb to web
plumb start web $data

# urls go to web browser
type is text
data matches '(https?|ftp|file|gopher|mailto|news|nntp|telnet|wais|prospero)://[a-zA-Z0-9_@\-]+([.:][a-zA-Z0-9_@\-]+)*/?[a-zA-Z0-9_?,%#~&/\-+=]+([:.][a-zA-Z0-9_?,%#~&/\-+=]+)*'
plumb to web
plumb start web $0

# man index entries are synthesized
type is text
data matches '([a-zA-Z¡-￿0-9_\-./]+)\(([1-8])\)'
plumb start rc -c 'man '$2' '$1' >[2=1] | nobs | plumb -i -d edit -a ''action=showdata filename=/man/'$1'('$2')'''


# lib/plumbing
# to update: cat $HOME/lib/plumbing.b | 9p write plumb/rules
# and cat $HOME/lib/plumbing | 9p write plumb/rules
editor = acme
include fileaddr

# postscript/pdf/dvi go to page but not over the a plumb port
# the port is here for reference but is unused
type is text
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.(pdf|PDF)'
arg isfile	$0
plumb to mupdf
plumb start mupdf $file

# open office - s[xt][cdigmw], docx?, xlsx?, pptx?
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.([Ss][XxTt][CcDdIiGgMmWw]|[Dd][Oo][Cc][Xx]?|[Xx][Ll][Ss][Xx]?|[Pp][Pp][Tt][Xx]?)'
arg isfile	$0
plumb to openoffice4
plumb start openoffice4 $file

# image files go to mupdf
type is text
data matches '[a-zA-Z¡-￿0-9_\-./]+'
data matches '([a-zA-Z¡-￿0-9_\-./]+)\.(jpe?g|JPE?G|gif|GIF|tiff?|TIFF?|ppm|bit|png|PNG)'
arg isfile	$0
plumb to mupdf
plumb start mupdf $file

# relative files as file: urls get made into absolute paths
type is text
data matches 'file:([.a-zA-Z¡-￿0-9_\-]([.a-zA-Z¡-￿0-9_/\-]*[a-zA-Z¡-￿0-9_/\-]))?'
arg isfile	$1
data set	file://$file
plumb to web
plumb start web $data

# urls go to web browser
type is text
data matches '(https?|ftp|file|gopher|mailto|news|nntp|telnet|wais|prospero)://[a-zA-Z0-9_@\-]+([.:][a-zA-Z0-9_@\-]+)*/?[a-zA-Z0-9_?,%#~&/\-+=]+([:.][a-zA-Z0-9_?,%#~&/\-+=]+)*'
plumb to web
plumb start web $0

# man index entries are synthesized
type is text
data matches '([a-zA-Z¡-￿0-9_\-./]+)\(([1-8])\)'
plumb start rc -c 'man '$2' '$1' >[2=1] | nobs | plumb -i -d edit -a ''action=showdata filename=/man/'$1'('$2')'''




