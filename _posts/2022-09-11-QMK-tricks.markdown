---
title: QMK Layer Issues and Troubleshooting
tags: [QMK, Mechanical Keyboards, Programming, Keyboards]
categories: [Programming]
---

## Overview

Quantum Mechanical Keyboard (or [qmk](https://qmk.fm/) ) is an amazing software library that lets you program your mechanical keyboard in all sorts of ways. 

However, I've had some 'minor' issues in trying to get my keyboard to do *exactly* what I want, and it took lots of time combing through the [documentation](https://docs.qmk.fm/) to figure out the solutions to my issues. So I made this 'Layer Issues and Troubleshooting' page to document some of my techniques and best-practices to solve some of these problems.

***

## Issue: Mod-tap or Layer-tap can't send shifted character

Let's say you have a dual-function key. On tap, it produces, say, the letter `a`. On hold, it temporarily applies a new layer. This works as expected with the `LT()` function. But, if you want it to send a modified character like `(` (which is really `9` but with a shift modifier) it can't do it. It only works on 'simple' codes.

### Solution: override the custom key handling

There's a great function called `process_record_user` that will handle a keypress in a customized way.

In this case, you want to add to this function (or create it if it doesn't exist) in your `keymap.c` file. 

If you wanted a key to produce `(` on tap and toggle layer 1 on hold, you would call it `LT(1, KC_LPRN)` and have a function like this: 

```c
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
  switch (keycode) {
    case LT(1,KC_LPRN):
        if (record->tap.count && record->event.pressed) { //normally the tap sends nothing, you intercept it here
            tap_code16(KC_LPRN); 
            return false; // stop handling this event
        }
    break;
```

This means that on tap, this function taps `KC_LPRN` and then stops processing the key press.

***

## Issue: Mod-tap can't switch to a layer while applying a modifier to that layer

Let's say you have a numbers layer, and rather than creating a symbols layer, you just want to press a button that layer-taps you into the numbers but with 'shift' held down at the same time.

There's no default way to do this in QMK. You can switch to a new layer, but you can't switch to the layer with a modifier held down simultaniously.

### Solution: add the mod to the key handler

This looks similar to the solution above, and you will put this in the same `switch` statement as above.

The critical difference is returning `true` instead of `false` so the key press gets processed further. 

In this case, say you wanted a key to temporarily toggle Layer 1 with the left shift modifier pressed on hold, and produce a comma (,) when tapped:

```c
   case LT(1, KC_COMM):
        if(record->event.pressed){ //pressed down
            if(!record->tap.count){ //hasn't had a tap yet, so just being held down
                register_mods(MOD_LSFT);
                }
        }
        else{
            unregister_mods(MOD_LSFT); // key up
        }
        return true; // keep processing key!
        break;
```
Now, holding this key down and another key will cause the second key to output its shifted version in layer 1.

***

## Issue: You want a fully customized behavior on tap and hold of a particular key

Say you want a key to copy on tap and paste on hold.

### Solution: it's in the docs 🙂

This solution is actually in [the documentation](https://github.com/qmk/qmk_firmware/blob/master/docs/mod_tap.md#changing-both-tap-and-hold). 🙂

So why bring it up? With the popularity of the [Oryx configurator](https://configure.zsa.io) many people will try to use the advanced tap-dance configurations to produce this behavior instead. While it's not wrong and will *probably* work, I've found it to be a little difficult to trigger reliably. In general, I've found using mod-tap works best and the 'advanced' tap dance configurations should only be used when you need to do something truly advanced.

***

## Issue: auto-shift doesn't shift 'complex' key codes

Auto-shift lets you (as the name implies) auto-shift keys. So, if you quickly tap 's', you'll get 's'. Hold it down a bit longer and you get 'S'. It's nifty and works fine if your key is sending `KC_S` but if your key is actually something like `MT(MOD_LALT, KC_S)`, then it won't work and you might bang your head on the table endlessly figuring out why. 

### Solution: define custom autoshift behavior

This is in the docs, but not totally clear or obvious (as of this writing)

First, you have to understand the idea of 'retro tapping'. 

The idea behind this is that if you did have a key like `MT(MOD_LALT, KC_S)` above, it won't ever send a 's' if you hold it down past the tapping term. It'll send a 's' on quick tap and act like you're holding down the Alt key otherwise. If you want to hold it down and get a 's', you have to enable 'retro tap'.

In terms of autoshift, basically what you want to happen is that if you hold this key down for some quick amount of time, you get a 's', some other amount of time and you get a 'S', and a different amount of time and it assumes you're pressing 'alt'. There's some fine-tuning on the timings to get it right, but in terms of auto-shift world, this behavior makes this key a 'retro' key because it requires 'retro tapping' to work.

The simplest way to get it running is to:

1. define a `RETRO_SHIFT` term in your `config.h` file. Contrary to intuition, this is actually the *timeout*, after which the given 'retro' keys will only produce their hold values, so it's a good idea to set it kind of high (like 400)

2. Create a function like below:

```c
bool get_custom_auto_shifted_key(uint16_t keycode, keyrecord_t *record) {
    if (IS_RETRO(keycode)){return true;}
    else {
      return false;
    }
}

```

This function will actually allow you to set whether autoshift behavior is defined on a per-key basis. The 'retro' keys are outside the standard autoshift ranges, so, again, you will need this if you're not sending a simple key code.
