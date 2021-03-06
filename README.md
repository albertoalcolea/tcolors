# tcolors

[![Build Status](https://github.com/albertoalcolea/tcolors/workflows/Tests/badge.svg)](https://github.com/albertoalcolea/tcolors/actions?query=workflow%3ATests)
[![Latest PyPI Version](https://img.shields.io/pypi/v/tcolors.svg)](https://pypi.python.org/pypi/tcolors)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/tcolors.svg)](https://pypi.python.org/pypi/tcolors)

Simple and easily-disableable library to make your life easier when working with ANSI colors in the terminal.

Without dozens of options, methods and combinations. Coloring the terminal should be a simple task.

Supports basic colors, bright colors, 256 ANSI colors, true colors and most of the ANSI text modifiers.

## Installation

Install *tcolors* by using pip:

```
$ pip install tcolors
```

## Motivation

There are dozens, if not hundreds, of libraries that color the terminal with ANSI escape sequences, why develop another one?

Mainly because I always miss a feature in almost all existing libraries: the option to enable or disable the colored automacally.

This is especially useful when developing command line applications whose output may be *piped* into another. In such cases it is quite practical to have a flag to globally disable coloring (`--no-colors` for example) and make it easier for downstream applications that don't have to deal with data polluted by escape sequences.

## Usage

### Colors

*tcolors* can color text with different color palettes. Take into account that not all terminal emulators support all of them.

**Basic 3-bit colors**

| Color   | Foreground    | Background      |
|---------|---------------|-----------------|
| Black   | Color.BLACK   | BgColor.BLACK   |
| Red     | Color.RED     | BgColor.RED     |
| Green   | Color.GREEN   | BgColor.GREEN   |
| Yellow  | Color.YELLOW  | BgColor.YELLOW  |
| Blue    | Color.BLUE    | BgColor.BLUE    |
| Magenta | Color.MAGENTA | BgColor.MAGENTA |
| Cyan    | Color.CYAN    | BgColor.CYAN    |
| White   | Color.WHITE   | BgColor.WHITE   |

**Bright colors**

| Color   | Foreground           | Background             |
|---------|----------------------|------------------------|
| Black   | Color.BRIGHT_BLACK   | BgColor.BRIGHT_BLACK   |
| Red     | Color.BRIGHT_RED     | BgColor.BRIGHT_RED     |
| Green   | Color.BRIGHT_GREEN   | BgColor.BRIGHT_GREEN   |
| Yellow  | Color.BRIGHT_YELLOW  | BgColor.BRIGHT_YELLOW  |
| Blue    | Color.BRIGHT_BLUE    | BgColor.BRIGHT_BLUE    |
| Magenta | Color.BRIGHT_MAGENTA | BgColor.BRIGHT_MAGENTA |
| Cyan    | Color.BRIGHT_CYAN    | BgColor.BRIGHT_CYAN    |
| White   | Color.BRIGHT_WHITE   | BgColor.BRIGHT_WHITE   |

**Modifiers**

| Modifier      | Description                                                |                 |
|---------------|------------------------------------------------------------|-----------------|
| Reset         | Reset all attributes                                       | Mod.RESET       |
| Bold          | Bold                                                       | Mod.BOLD        |
| Dimmed        | Faint, decreased intensity, or dim                         | Mod.DIM         |
| Italic        | Italic                                                     | Mod.ITALIC      |
| Underline     | Underline                                                  | Mod.UNDERLINE   |
| Slow blink    | Blink less than 150 times per minute                       | Mod.SLOW_BLINK  |
| Rapid blink   | Blink more than 150 times per minute. Not widely supported | Mod.RAPID_BLINK |
| Inverted      | Swap foreground and background colors                      | Mod.INVERT      |
| Conceal       | Conceal or hide. Not widely supported                      | Mod.CONCEAL     |
| Strikethrough | Strikethrough                                              | Mod.STRIKE      |

**256 colors**

| Color | Foreground     | Background       |
|-------|----------------|------------------|
| N     | Color256.C_{N} | BgColor256.C_{N} |

Where `{N}` can be any number from 0 to 255.

**True colors**

There are no built-in colors for the more than 16M of colors of the true color palette, but you can create your own colors just by creating a new instance of `TrueColor` or `BgTrueColor`

```python
from tcolors import TrueColor, BgTrueColor

# Define colors either as hex strings (normal or short) or as rgb values
BLUE = TrueColor('#1F2041')
PURPLE = TrueColor('#437')
GREEN = TrueColor(65, 123, 90)
BG_COLOR = BgTrueColor(r=63, g=123, b=90)
```

### Colorize text

You can add color in two ways:

- `colorize`: given a string it return a new string with the color escape sequences.
- `cprint`: print applying colors to the printed text.

`cprint` has the same signature as the built-in print function with an extra keyword argument `style` to define the style to apply.

```python
from tcolors import Color, Mod, colorize, cprint

colorize('Text with colors', Color.RED)
cprint('Print with color', style=Mod.BOLD)
```

Both methods adds the reset modifier at the end automatically.

In fact you can concatenate styles (all colors and modifiers are instances of the Style class) to any string directly. Just remember that in these cases it is necessary to put the reset modifier manually.

```python
print(Color.RED + 'Colored text' + Mod.RESET)
print(Color.RED, 'Colored text', Mod.RESET, sep='')

str1 = 'This is a {}blue{} text'.format(Color.BLUE, Mod.RESET)
str2 = 'This is a %syellow%s text' % (Color.YELLOW + Mod.BOLD, Mod.RESET)
str3 = f'{Color.RED}{Mod.BOLD}This is a text in bold red{Mod.RESET}'
```

### Mix styles together

It is possible to combine styles using the addition operator.

```python
cprint('This is a warning', style=Color.RED + BgColor.YELLOW + Mod.BOLD + Mod.Italic)
```

If you plan to use the same combination frequently you can assign it to a variable:

```python
title_style = Color.BLUE + Mod.UNDERLINE
cprint('Title 1', style=title_style)
colorize('Another title', style=title_style)
```

### Configuration

You can configure *tcolors* to enable or disable the colored globally.

If it is disabled all calls to `colorize` or `cprint` won't color the text.

```python
from tcolors import configure_colors

configure_colors(enable_colors=False)
```

You can also define a default style that will be added automatically in all calls to `colorize` or `cprint`

```python
from tcolors import configure_colors, Mod

# Put everything in bold
configure_colors(default_style, Mod.BOLD)

cprint('Just an example...', style=Color.GREEN)  # It will be printed in green and bold
cprint('Only bold')
```

### Colorizer class

If you need to colorize several things in a different way use custom instances of the `Colorizer` class.

```python
from tcolors import Colorizer, Mod

colorizer1 = Colorizer(default_style=Mod.BOLD)
colorizer2 = Colorizer(default_style=Mod.ITALIC)

colorizer1.colorize('The colorizer1 instance always prints text in bold', style=Color.BLUE)
colorizer2.colorize('While colorizer2 always prints in italic', style=Color.YELLOW)
```

As you can see the behaviour is the same as in the `colorize` and `cprint` functions. That is because *tcolors* uses an instance of the `Colorizer` class internally and exports those methods. Combine custom instances with the internal instance as you will.

If you want to use some default style only in a certain piece of code you can use `Colorizer` as a context manager:

```python
from tcolors import Colorizer

with Colorizer(default_style=Color.GREEN) as c:
    c.cprint('Inside the context manager')
```

### Examples

You can find more practical examples in the `examples` directoty.
