---
title: Replace Caps Lock with Backspace functionality
layout: post
---

Tired of a complete miss use of one of the biggest keys on keyboard, and inaccessibility of the most important.

```sh
xmodmap -e 'remove Lock = Caps_Lock'
xmodmap -e 'keycode 66 = BackSpace'
xmodmap -pke > .mykeymap
```

