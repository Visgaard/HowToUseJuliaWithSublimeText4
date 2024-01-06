# How to use Julia with Sublime Text 4 (done on build 4169)

## Rationale

This document is copied directly from the original branch and I have made a few corrections. I have done absolutely no additional work, and everthing should be attributed to [@PetrKryslUCSD](https://www.github.com/PetrKryslUCSD). I've used this to get a nice Sublime Text setup with a Julia REPL inside the editor, to which we can send code and build source code.

## Installation

### Sublime Text 4

In the following we describe how to set up the editor from scratch. Download
the editor from the website [https://www.sublimetext.com/](https://www.sublimetext.com/). As I am on Windows 11, I will describe the installation and use of the
editor on that platform.

### Install Package Control

The first thing you would want to do is install the package **Package Control**.
Use the menu item **Tools/Command Palette** (key binding: `ctrl+shift+P`), type `install`, and an item **Install Package Control** will bubble up. Select it, and practically instantaneously you will get a confirmation box that the package was installed.

By the way, remember the key binding `ctrl+shift+P`: it will bring up the **Command Palette**, and practically everything that you might need can be done from here.

### Install Julia syntax highlighting

**Package Control** can now be used to install other useful packages.
Bring up the **Command Palette**, and type in a few letters of **Package Control: Install Package**. When it comes up, select it. Loading the registry will take a couple of seconds, and then we get window where we can type in `Julia`. A button named **Julia** (with the subtitle "Julia syntax highlighting for Sublime Text 4"; refer to [https://github.com/JuliaEditorSupport/Julia-sublime](https://github.com/JuliaEditorSupport/Julia-sublime)) will come up highlighted. Click on it, and the package will be installed. At this point one should be able to open a Julia source file and get it highlighted based upon the syntax of Julia.

### Install the Terminus and SendCode packages

Repeat the procedure of package installation from **Package Control** to install the terminal emulation package **Terminus** (https://github.com/randy3k/Terminus), and the package to enable communication between a source window and the terminal, **SendCode** (https://github.com/randy3k/SendCode).

## Customization

The key bindings file and the other customization files for the user live in the folder for user settings. In my case that is `C:\Users\Rvisg\AppData\Roaming\Sublime Text\Packages\User`. For brevity I will call this folder `USER`.

### Basic user settings

Select **Preferences/Settings**. Sublime will open two windows on the left and right of your editor, both called Preferences.sublime-settings, where the left one is Sublime default settings and the right one is your user defined settings, which will be saved automatically in the `USER` folder. This is the file you need to make changes in, as the default settings gets overwritten with Sublime version updates. All possible settings are listed in the default settings, and you can always refer to these. I like the following settings: 

```
{
    "always_show_minimap_viewport": true,
    "auto_complete": false,
    "auto_complete_commit_on_tab": false,
    "draw_minimap_border": true,
    "draw_white_space": "all",
    "font_size": 12,
    "highlight_line": true,
    "ignored_packages":
    [
        "Vintage",
    ],  
    "index_files": true,
    "margin": 1,
    "preview_on_click": false,
    "translate_tabs_to_spaces": false,
    "word_wrap": true,
}

```

### Key bindings

Select **Preferences/Key bindings**. This will open a new window with
the file`Default (Windows).sublime-keymap`, shown in the window on the right. The same thing goes for this file as the user settings file.
I put in there my personal preferences for key bindings:

```
[
    // pasted code immediately re-indented
    { "keys": ["ctrl+v"], "command": "paste_and_indent" },
    { "keys": ["ctrl+shift+v"], "command": "paste" },
    
    // Access to code evaluation. A current line,
    // or a selection is passed to the terminal for evaluation
    // (the command belongs to the SendCode package).
    {
        "keys": ["ctrl+keypad_enter"], "command": "send_code",
            "context": [
               { "key": "selector", "operator": "equal", "operand": "source" }
            ]
    },
    
    // To make the copy and paste keys work in the Terminus window
    // (otherwise they are ctrl+shift+c, ctrl+shift+v)
    { "keys": ["ctrl+c"], "command": "terminus_copy",
        "context": [
            { "key": "terminus_view" },
            { "key": "terminus_view.natural_keyboard" },
            { "key": "selection_empty", "operator": "equal", "operand": false, "match_all": true }
        ]
    },
    { "keys": ["ctrl+v"], "command": "terminus_paste",
        "context": [
            { "key": "terminus_view" },
            { "key": "terminus_view.natural_keyboard" }
        ]
    },
]

```

### Customization of the SendCode package

I make sure Julia code is sent to a **Terminus** terminal. Select **Preferences/Package Settings/SendCode/Settings**. Paste the below code into the file `USER` file

```

    "prog": "terminus",

    "julia" : {
        "prog": "terminus",
        "bracketed_paste_mode": false
    }

```
There also needs to be a file `Packages\SendCode\build_systems\Julia - Source File.sublime-build` with the command to "build" a Julia file by running it in the REPL. This file should have been created when you installed **SendCode**. 
```
{
    "target": "send_code_exec",
    "code": "include(\"$file_name\")",
    "selector": "source.julia"
}
```

I replaced the existing `"code": "include(\"$file\");",` with `"code": "include(\"$file_name\")",` to get just the file name instead of the full path to the file. Check the [SendCode repo](https://github.com/randy3k/SendCode) for other options.

### Command to start Julia REPL

The following functionality is the brainchild of Paul Soderlind (thanks!).

In order to be able to open a Julia REPL from a Julia source file currently opened in the editor, I define the following command binding in the file `USER\Default.sublime-commands`. You made need to create this file yourself. Remember the correct extension on the file!
```
[
    {
        "caption": "Terminus: Open Julia 1.9.3",
        "command": "terminus_open",
        "args"   : {//"shell_cmd": "%LOCALAPPDATA%/Programs/julia-1.9.3/bin/julia.exe",
            "shell_cmd": "%LOCALAPPDATA%\\Programs\\julia-1.9.3\\bin\\julia.exe",
            "cwd": "${file_path:${folder}}",
            "title": "Julia REPL",
            "env": {"JULIA_NUM_THREADS":"4"},
        }
    }
]
```
Note that I have manually put in the location of the julia executeable. If you are not sure where it is located, type in `Sys.BINDIR` in a Julia REPL. Furthermore, you can edit the caption as you like and call it what you want. You can duplicate the code above to create another terminus that uses a different version of Julia. So in the **Tools/Command Palette** choose the command **Terminus: Open Julia 1.9.3** 

Also, it is possible to set the environment variable directing the use of threading, `JULIA_NUM_THREADS`. Extending this to other environment variables is likely to be successful as well.

## Usage

### Open a source file and then run Julia from the source file

Open a Julia source file, and in the **Command Palette**, start typing in `Terminus`. Select **Terminus: Open Julia 1.9.3**. This will open the default terminal on the user's platform, and run Julia in the directory that contains the open file.

### Evaluating code

Select some Julia code and press `ctrl+enter`. The code will be pasted into the **Terminus** window and evaluated in Julia (assuming Julia was started in that **Terminus** window).
This also works for evaluating a line of code: place the cursor on a line and type `ctrl+enter`.

### Running Julia files

In a currently opened Julia source file, press `ctrl+b` (which is a key binding for the menu action **Tools/Build**). The current file will be evaluated in a Julia-running **Terminus** window with an `include()`.

If you need to restart a session in the Julia REPL, you have to close the terminal and open a new one.
