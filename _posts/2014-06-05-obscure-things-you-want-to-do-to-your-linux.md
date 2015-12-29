---
title: Obscure things you want to do to your 'nix
layout: post
---

Tired of a complete misuse of one of the biggest keys on keyboard, and inaccessibility of the most important.

    setxkbmap -option ctrl:nocaps

    xmodmap -e 'remove Lock = Caps_Lock'
    xmodmap -e 'keycode 66 = BackSpace'
    
    xmodmap -pke > .mykeymap

Missing a certificate?

    mozroots --import --ask-remove

Restart sound..

    pulseaudio -k && sudo alsa force-reload

mount usb

    mount -o gid=1000,uid=1000 /dev/sdb1 usb