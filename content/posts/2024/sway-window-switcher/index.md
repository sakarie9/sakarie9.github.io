---
date: "2024-11-26T15:03:26+08:00"
title: "Sway Window Switcher"
tags: [Linux]
categories: 技术分享
---

{{< github repo="sakarie9/sway-window-switcher" >}}

A Python script to help list and switch sway windows easily through Fuzzel.

![Screenshot-1](screenshot-1.webp)

## Requirements

- Sway
- A dmenu launcher, including fuzzel and rofi
- Python3
- jq
- Grep

Also a nerd font is required if icons not show correctly. You can change fonts in your launcher settings.

## Usage

1. Get `sway-window-switcher.py` to somewhere you can execute.

2. Run `sway-window-switcher.py --help` to get help.

   ```bash
   > sway-window-switcher.py -h
    usage: sway-window-switcher.py [-h] [-t {all,floating,scratch,regular}] [--plain-output] [-l {fuzzel,rofi}]

    List and select windows using swaymsg and dmenu launchers like fuzzel and rofi.

    options:
      -h, --help            show this help message and exit
      -t {all,floating,scratch,regular}, --type {all,floating,scratch,regular}
                            Type of window to list. Defaults to "all".
      --plain-output        Print a plain, unbeautified list to dmenu.
      -l {fuzzel,rofi}, --launcher {fuzzel,rofi}
                        Specific a dmenu launcher to use. Will use first exist launcher if not define.
   ```

3. Bind the script to sway keybind to use it easily.

   For example, to list all the windows in scratchpad and switch to it you can have this in sway config:

   `bindsym $mod+x exec sway-window-switcher -t scratch`

   Make sure `sway-window-switcher` is in you PATH.

## Thanks to

- [codingotaku/fuzzel-scripts](https://codeberg.org/codingotaku/fuzzel-scripts)

  This script is basically a more featured python rewrite version of codingotaku's script.

- [Jas-SinghFSU/HyprPanel](https://github.com/Jas-SinghFSU/HyprPanel)

  We used the [windowTitleMap](https://github.com/Jas-SinghFSU/HyprPanel/blob/f4834ec308545d1ac27815e210e998f61c3435c8/modules/bar/window_title/index.ts#L15) from HyprPanel for a better look.
