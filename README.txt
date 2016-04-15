Fake "hicolor" icon set. Apr 15 2016

Hi.  Mike Gran here.

When I compiled gtk3 on windows using the https://github.com/wingtk/gtk-win32
method, it doesn't come with an icon set.  So it tries to
use the fallback "hicolor" icon theme.

But this fallback theme "hicolor" hasn't been used by Gnome or KDE for
a long time.  The both have complete icon sets that mask the need for a
fallback icon set.  Gnome has Adiwata.  KDE has many icon sets.  So for
both Gnome and KDE, they've removed all the icons from hicolor as
moot.

But until I can figure out how to build Adiwata via a gtk-win32 type method,
the hicolor icons are not moot. 

KDE classic is a direct descendent of the original hicolor theme, so it
is pretty close in spirit to the original hicolor.

Also it has the advantage of being very unlike the GNOME Adiwata theme, so
you can tell when GTK3 is using a fallback icon.

So, this "hicolor" icon theme is actually culled from KDE Classic icons at
svn://anonsvn.kde.org/home/kde/tags/KDE/4.9.98/kdeartwork/IconThemes/kdeclassic

But the icon set from the above location doesn't have the folders as
expected in freedesktop.org's definition of what hicolor is.
Freedesktop's folder list is hicolor-icon-theme-0.15 as it appears on 
https://www.freedesktop.org/wiki/Software/icon-theme/

So a couple of folders were removed to make the icon set from
KDEClassic to be a subset of the structure as expected in
hicolor-icon-theme-0.15.

Note that there are many sizes and categories of icons in this icon
set that show up in hicolor-icon-theme-0.15 that do not exist here.

Aside from this one note and some empty directories, all of this work
comes from KDE.  Note the "LICENSE" file for legal information
on these KDE icons.
