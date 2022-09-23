---
title: Getting KMonad working in OSX with a non-US keyboard
date: 2022-09-21
categories: [Programming]
tags: [Keyboards, MacOS, tinkering]
---

## KMonad vs QMK

As I've discussed before, [qmk](https://qmk.fm/) is an amazing piece of software designed to program your mechanical keyboard. The problem, of course, is that you have to have a mechanical keyboard to use it.

But, what if, instead, you could run software on your computer itself that would re-interpret keypresses on your regular keyboard to give it some of the 'magic' that QMK has?

Enter [KMonad](https://github.com/kmonad/kmonad)

![kmonad-logo](https://github.com/kmonad/kmonad/raw/master/kmonad.svg){: width="40%" }

This nifty software lets you do simple things like key remappings, along with fancier things like layer-taps and tap-dances with any keyboard (provided it's connected to a computer running KMonad).

## When to prefer it

Of course, it doesn't make sense to use if you have a programmable mechanical keyboard already, but consider the built-in keyboard in your laptop.

For me personally, a huge advantage of my keyboard layout is that I have backspace and return close to my thumbs, as opposed to having them somewhere in the corner where my right pinkie would need to stretch to hit them.

Just moving these keys somewhere close to the spacebar would make my laptop keyboard much more usable. Similarly, if you've gotten accustomed to home-row mods (modifier keys activated in your home row), you can use this to get the same behavior natively out of your laptop keyboard.

## Setting it up on OSX with a non-US keyboard

It should be as simple as following along on the github page, right? Well, I hit a few snags, so I wrote this to help anyone else stuck as well

### Download the code

Go to the [github site](https://github.com/kmonad/kmonad) and clone the code into a directory

### Install 'dext' component 

The key input-grabbing mechanism is actually taken from [Karabiner Elements](https://karabiner-elements.pqrs.org/). As of this writing, KMonad is dependent on a specific version of the 'dext' library component from Karabiner Elements, meaning **it will not work together with Karabiner**. This is important if (like me) you used Karabiner for simple key remappings.

Here is how you make sure you install the correct `dext` library (from install docs):  
```console
  $ git clone --recursive https://github.com/kmonad/kmonad.git
  $ cd kmonad/
  $ open c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/dist/Karabiner-DriverKit-VirtualHIDDevice-1.15.0.pkg
  $ /Applications/.Karabiner-VirtualHIDDevice-Manager.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Manager activate
```

### Dealing with the § key if you have it

From the [github issue](https://github.com/kmonad/kmonad/issues/106) page on this

If you have a non-US keyboard, you may have an extra key that is sometimes called a 'Non-US backslash'. On OSX, if you switch your key layout to US, this key will type out a § symbol.

If you have this key and press it while KMonad is running, it will crash irrecoverably 😢.

You can fix this by going into the directory you have cloned into above (we'll call it `kmonad` for consistency), and editing the following file:

`kmonad/src/KMonad/Keyboard/IO/Mac/Types.hs`

Find the line that says:

```haskell
-- , ((0x7,0x64), KeyNonUSBackslash)
```

and un-comment it to say something like the following

```haskell
, ((0x7,0x64), KeyF13)
```

You've now backed the § key to F13 (which most computers don't have). This won't really do anything except:
1. When KMonad is running, your computer will interpret a key press here as pressing `F13`

2. If referencing this key inside the `defsrc` section of your KMonad keyboard mappings, you have to call it `f13`

### Compile with Stack

Because this is written in Haskell, you'll need to use [Stack build](https://docs.haskellstack.org/en/latest/build_command/) to build it. If it's not installed, install it first.

```console
$ brew install stack
```
**Important**: be sure to run the following from *inside the kmonad/* directory:
```console
$ stack build --flag kmonad:dext --extra-include-dirs=c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/include/pqrs/karabiner/driverkit:c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/src/Client/vendor/include
```

### Move the binary

At the end of the compilation, stack will give the location that it build the binary (just called `kmonad`)-- move the binary somewhere easier to find and remember (like `~/bin/`)

### Set privacy preferences

You have to specifically allow the `kmonad` binary to listen to all key presses and intercept them.

Navigate to `System Preferences > Security & Privacy > Privacy > Input Monitoring` and add the `kmonad` binary from the location you created **AND** `Terminal.app` (for now) to allow them to accept keypresses

**NOTE: if you get some sort of permission or connect error at this stage, it could be an incompatible version of Karibeaner**

### Create a keyboard config and test from the command line

Create a keyboard config file (or just use the [default mac](https://github.com/kmonad/kmonad/blob/master/keymap/template/apple.kbd) config).

Assuming you copied it to the same directory as the `kmonad` binary, run it **as sudo** inside the built-in Terminal app:

```console
#sudo ./kmonad apple.kbd
```
Type around a bit, if you don't get any errors, it should all be working fine.

Exit with `Ctrl-C`

### Launch Kmonad on startup

Slightly modified from the [issue page](https://github.com/kmonad/kmonad/issues/105) 

Create the following file named `local.kmonad.plist`, changing values of `/path/to/kmonad/` and `/path/to/kbd/file.kbd/` to the correct locations.

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>local.kmonad</string>
    <key>Program</key>
    <string>/path/to/kmonad</string>
    <key>ProgramArguments</key>
    <array>
      <string>/path/to/kmonad</string>
      <string>/path/to/kbd/file.kbd</string>
    </array>
    <key>RunAtLoad</key>
    <true />
    <key>StandardOutPath</key>
    <string>/tmp/kmonad.stdout</string>
    <key>StandardErrorPath</key>
    <string>/tmp/kmonad.stderr</string>
  </dict>
</plist>
```

Move this plist file to `/Library/LaunchDaemons/`

**NOTE: the binary and the .kbd file will need to be readable by launchctl's default user. To play it safe, I gave both of these files universal read and execute permissions**

### Final permissions

Grant input changing permissions to the launchtl binary
(`/bin/launchctl`) using the same `System Preferences` menu as described before.

Unfortunately, you can't just type the path in directly since the input-changing permissions library has a graphical user interface.

To get it in there, go to your Desktop, press `Cmd+shift+G`, and then `/` into the prompt window that comes up. You'll now have the root directory (`/`) loaded into a Finder window. Now you can drag the `launchctl` binary in to the input-changing permissions menu.

## That's it, kinda

Now you can run KMonad, hopefully. There are some pretty big quirks around its syntax, how it works, etc. Conceptually, it lets you do the same kinds of things that QMK does, but the way its syntax is formed and the concepts behind it are a little different and take a while to wrap your head around.

But it's worth it — your fingers will thank you!