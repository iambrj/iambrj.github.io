---
layout: post
title: "Vim as a debugger"
author: Bharathi Ramana Joshi
categories: tips
tags: [vim]
image:
---

I recently learned about vim's `termdebug` feature and this is small tutorial on
the same. This requires at least vim 8.0 and a working gdb.

I shall be demonstrating it on the following C++ program.

```
#include <iostream>

int factorial(int n)
{
    std::cout << "n = " << n << std::endl;
    if(n < 2)
        return 1;
    else
        return n * factorial(n - 1);
}

int main()
{
    int n = 5;
    int fact = factorial(n);
    std::cout << fact << std::endl;
}
```

Start vim, copy the above code into it and save the file as, say

```
factorial.cpp
```

To enable the debugger interface, enter

```
:packadd termdebug
```

in normal mode. For regular use, just add this to your vimrc.

To start the interface, enter

```
:Termdebug
```

in normal mode. Now, you should see the following
(perhaps with a different split layout).

![https://user-images.githubusercontent.com/29101670/79291347-06d18f00-7eec-11ea-913a-5196609f7432.png](https://user-images.githubusercontent.com/29101670/79291347-06d18f00-7eec-11ea-913a-5196609f7432.png)

One window is the source window, one is a gdb shell and the other is for
interacting with program loaded.

Before proceeding further, I recommend `:set mouse=a` so that the mouse can be
used to interact with vim.

Now compile the code by entering

```
:!g++ -g % -o factorial
```

in normal mode. Go to the gdb window and enter

```
file factorial
```

to load the executable.

Now you can not only use gdb in text mode as usual, but also use it from vim! For instance,
try moving your mouse to some line and press right click -> set breakpoint. Note
that this requires `:set mouse=a`, if you haven't done this you can do a

```
:Break
```

after moving the cursor to the required line.

To start executing the program either do it from gdb as

```
run factorial
```

or from vim as

```
:Run
```

You can use the buttons at the top of the source window to interactively debug the
program. As you should see something as follows.

![https://user-images.githubusercontent.com/29101670/79291172-932f8200-7eeb-11ea-9cea-a0fe9703cfa8.png](https://user-images.githubusercontent.com/29101670/79291172-932f8200-7eeb-11ea-9cea-a0fe9703cfa8.png)

For a comprehensive explanation of what can be done see
[this](https://vimhelp.org/terminal.txt.html#terminal-debug) or enter `:help terminal-debug`.
