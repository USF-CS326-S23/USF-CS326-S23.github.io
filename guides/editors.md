---
layout: default
title: Editors
nav_order: 1
parent: Guides
permalink: /guides/editors
---

# Working with Console-Based Editors

Getting comfortable with a console-based editor is important in many software development situaitons, like when working on remote cloud services or smaller single-board computers that are not running a GUI. While using GUI-based editors like Sublime and Visual Studio Code and IDEs like IntelliJ and Eclipse can be very productive, working with and editor directly from the command like can be equally productive. In fact, in my view you should be able to work in all three types of editors: GUI, IDE, and console. Three major console-based editors are micro, vim, and emacs. They are all fairly powerful and can do many of the things you find in GUI-based editors like displaying line numbers, support for multiple tabs, search and replace, code commenting etc. In this guide I will show some customizatoins you may want to add to micro and vim, especially for coding in xv6.

# Contents
1. [Micro](#micro)
2. [Vim](#vim)


## Micro

You can learn more about micro and how to install it here:

[Micro Website](https://micro-editor.github.io)

We have micro installed on griffin, but you may also want to install it locally on your machine or other lab machines. The nice thing about micro is that it is a single executable file and you can put it directly into your ```~/.local/bin``` directory for use.

Below I show the customizations I'm using with micro. You can customize micro by editing files in the ```~/.local/config/micro``` directory. Note that you have to run micro once for this directory and its files to be created.

### Config ```~/.config/micro/settings.json```

```json
[benson@griffin micro]$ cd ~/.config/micro
[benson@griffin micro]$ cat settings.json
{
    "clipboard": "internal",
    "colorscheme": "xcodelike",
    "ft:c": {
        "tabsize": 2
    },
    "tabstospaces": true
}
[benson@griffin micro]$
```

Unfortunately, when you use micro over ssh, working with the copy/paste clipboard can be a bit awkward. I've found the best experience is to use the "internal" clipboard, which means you can use CTRL-C (copy), CTRL-X (cut), and CTRL-V (paste). However, this will not work directly with the clipboard on your OS, which means you can't copy to your local clipboard with something like COMMAND-C. Most of the time, this is not an issue.

I've been using a custom colorscheme called "xcodelike." See below on how to add this colorscheem. You want to set tabstospaces to true so that you get spaces for indentation and not TAB characters. In this case, I've restricted the tabsize to 2 for C programs only ("ft:c"). This will be useful for following the xv6 formatting conventions.

### Colorscheme: ```~/.config/colorschemes/xcodelike.micro```

To use the xcodelike colorscheme you need to create ```colorsschemes/xcodelike.micro```:

```text
[benson@griffin micro]$ cd ~/.config/micro
[benson@griffin micro]$ ls
backups  bindings.json  buffers  colorschemes  settings.json
[benson@griffin micro]$ cd colorschemes/
[benson@griffin colorschemes]$ ls
xcodelike.micro
[benson@griffin colorschemes]$ cat xcodelike.micro
```
```
color-link default "#202020,#FFFFFF"
color-link comment "#009900"
color-link identifier "#6600CC"
color-link constant "#6600CC"
color-link constant.string "#CC0000"
color-link constant.string.char "#BDE6AD,#282828"
#color-link constant.string.url "#BDE6AD,#282828"
color-link statement "#0000CC"
color-link preproc "#0000CC"
color-link type "#0000CC"
#color-link special "#A6E22E,#282828"
color-link special "#0000CC"
#color-link underlined "#D33682,#282828"
color-link error "bold #CC0000"
color-link todo "bold #CC0000"
color-link statusline "#808080,#E0E0E0"
color-link tabbar "bold #808080,#E0E0E0"
color-link indent-char "#505050,#282828"
color-link line-number "#808080,#E0E0E0"
color-link current-line-number "#AAAAAA,#282828"
color-link diff-added "#00AF00"
color-link diff-modified "#FFAF00"
color-link diff-deleted "#D70000"
color-link gutter-error "#CB4B16,#282828"
color-link gutter-warning "#E6DB74,#282828"
color-link cursor-line "#99CCFF"
color-link color-column "#323232"
color-link symbol "#202020,#FFFFFF"
color-link symbol.tag "#AE81FF,#282828"
```

### Keybindings: ```~/.config/micro/bindings.json```

It is useful to read about the default micro keybindings [here](https://github.com/zyedidia/micro/blob/master/runtime/help/keybindings.md). Alternatively, you can type CTRL-g in the micro editor to get a help pane that gives an overview of the keybindings.

You can add or change keybindings by modifying the ```bindings.json``` file. I've changed the keybindings to work a bit more like Emacs:

```
[benson@griffin ~]$ cd ~/.config/micro
[benson@griffin micro]$ ls
backups  bindings.json  buffers  colorschemes  settings.json
[benson@griffin micro]$ cat bindings.json
```
```json
{
    "Alt-a": "SelectAll",
    "Alt-e": "CommandMode",
    "Ctrl-a": "StartOfLine",
    "Ctrl-e": "EndOfLine",
    "Ctrl-j": "JumpLine",
        "Ctrl-r": "HSplit",
    "Ctrl-y": "Paste",
    "CtrlDown": "CursorPageDown",
    "CtrlLeft": "PreviousTab",
    "CtrlRight": "NextTab",
    "CtrlShiftDown": "CursorEnd",
    "CtrlShiftUp": "CursorStart",
    "CtrlUnderscore": "lua:comment.comment",
    "CtrlUp": "CursorPageUp",
    "Alt-/": "lua:comment.comment",
    "CtrlUnderscore": "lua:comment.comment"
}
```

In micro you can type CTRL-T to create a new tab. I've modified the behavior of CTRL-LeftArrow and CTRL-RightArrow to cycle through the tabs. This makes it very easy to switch between tabs.


## Vim

[Vim](https://www.vim.org) stands for Vi Improved and is an extension to the original vi editor. Vim is a superset of the standard vi commmands and the vi editor (or vim) will be installed on almost any Unix/Linux system you interact with. So, even if you use micro, know some basic vi/vim commands will be useful.

Here is a very basic ```~/.vimrc``` file that will use spaces instead of tabs for indentation and use 2 spaces for indentation for C and C++ files. It also globally uses spaces instead of tabs and 4 spaces for other types of files. It turns on syntax highlighting and line numbers on the left side of the file.


### Vim configuration: ```~/.vimrc```
```
[benson@griffin ~]$ cd
[benson@griffin ~]$ cat .vimrc
```
```
syntax on
set number

" tabstop: Width of tab character
" softtabstop: Fine tunes the amount of white space to be added
" shiftwidth Determines the amount of whitespace to add in normal mode
" expandtab: When on uses space instead of tabs
set tabstop=4
set softtabstop=4
set shiftwidth=4
set expandtab

filetype on
au FileType c,cpp setlocal softtabstop=2 shiftwidth=2 tabstop=2 expandtab
```

