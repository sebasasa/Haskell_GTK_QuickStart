# Haskell GTK Quick Start

This is little more than a quick proyect i build to acts as a template for when i need to work with GUI in Haskell

I made this mainly because setting this up in my Mac was a BIG pain. And from now on i can simply come here to look for a simple repo that i can clone a some commands that i can copy paste to get everything working as fast as posible. I left it public because it might be helpful for some people. Might even make a tutorial about why setting up this was hard and how Haskell integration with C/C++ works in general in the future.

## Let's get started

Currently this instructions are only for **MacOS** so this is hardly a comprehensive list, but this is because this was the only enviroment i had trouble with, Windows has some very detailled instructions floating aroudn and linux suposedly just works out of the box.

### Setting up pkg-config

The first thing we should do is make sure this command finds something:

```bash
pkg-config --cflags gtk+-2.0
```

If it does, that means that everything is working and the GTK library is available in your system in a place that your linker will be able to find
If it doesn't that means either a problem with your pkg-config setup, or it means that GTK is not instlaled properly, so let's take care of both using brew.

For starters install pkg-config using brew, and then, make sure that's the one it being used by your system

```
brew install pkg-config
```

Now run this:

```
which -a pkg-config 
```

If the top of the list is the pkg-config installed by brew that means we are good to go, in my machine the path was this `/usr/local/bin/pkg-config`

If it is not, you have to handle that by hand my making the necesary adjustments in your path, or by removing the other alternatives so that the correct one stays at the top

Ok, now, let's install gtk, i will just dump here large amount of commands i pasted in my commandline while searching arround for solutions, running all of this should do the trick

```
brew install gtk+
brew install glib cairo gtk gettext fontconfig
brew deps gtk # see below
brew deps gtk | xargs brew link
brew link cairo gettext fontconfig
brew install gobject-introspection gtk+3
brew install cairo
brew install fontconfig
brew install bzip2
brew install expat
brew install zlib
brew install libpng

```

^ A lot of these commands came from this guide, so check them out in case they might be usefull
https://gist.github.com/peterjoel/3119341


Luego, siguiendo ese mismo tutorial tuve hacer lo de freetype. Y donde lo de freetype tuve que ejecutar todos los comandos como sudo
Habia que meter `/usr/local/opt/libffi/lib/pkgconfig` en la variable `PKG_CONFIG_PATH`. 

En fish simplemente hice esto:
```
set -Ux PKG_CONFIG_PATH /usr/local/opt/libffi/lib/pkgconfig
```

If none of that worked, the thing to look for in your desperate google search is `gtk+-2.0`. But `libgtk2.0-dev` o maybe just `libgtk2.0` might also yield useful results. As for pkg-config. The exact thing we are looking for is a file called `gtk+-2.0.pc` which should also be in the universal file `PKG_CONFIG_PATH`. Si quieres ir por ese enfoque y ver si ese archivo YA esta en tu path puedes hacer `locate '*.pc' | grep gtk`. En mi caso estaba aqui: `/usr/local/lib/pkgconfig`. Y simplemente me tocó hacer poner eso a mano en PKG_CONFIG_PATH. Resulthing in this:

```
set -Ux PKG_CONFIG_PATH /usr/local/opt/libffi/lib/pkgconfig /usr/local/lib/pkgconfig 
```
I hope that'  helpful in the case where you still were unable to get pkg-config to recognize `gtk+-2.0`


If, on the contrary, you now see something when you type `pkg-config --cflags gtk+-2.0` then you are good to go!

### Installing the libraries with cabal a

Now, let's continue with trying to see if cabal is ready to install gtk without complainign about beign unable to find something by running this command:

```
cabal install cabal-install
cabal install gtk -fhave-quartz-gtk --allow-newer=base gtk
```

In my machine this took a pretty long. But eventually they were done and instalation went succesfuly. So finally, let's set upt a project (This project) so that we can get to work.

### Creating the project

```
mkdir HaskellGUI
cd HaskellGUI
cabal init
```

That will create our proyect. Then let's edit `HaskellGUI.cabal`

```
```

And finally let's edit `app/Main.hs`

```Haskell
{-# LANGUAGE OverloadedStrings, OverloadedLabels #-}
{- cabal:
build-depends: base, haskell-gi-base, gi-gtk == 3.0.*
-}

import qualified GI.Gtk as Gtk
import Data.GI.Base

main :: IO ()
main = do
  Gtk.init Nothing

  win <- new Gtk.Window [ #title := "Hi there" ]

  on win #destroy Gtk.mainQuit

  button <- new Gtk.Button [ #label := "Click me" ]

  on button #clicked (set button [ #sensitive := False,
                                   #label := "Thanks for clicking me" ])

  #add win button

  #showAll win

  Gtk.main
```

With all this done you can finally do this (This command in my machine for some reason took about an hour, so be prepared):

```
cabal run HaskellGUI    
```

And after an awfully long wait, everything was, at last, functional (pun intended)

