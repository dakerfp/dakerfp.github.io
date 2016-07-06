+++
title = "How to set a X window visible or invisible using Xlib"
date = "2013-12-19"
tags = ["c", "c++", "visibility", "window", "X11", "Xlib"]
+++

```c++
bool m_visible;
Display* m_display;
Window m_window;

void setVisible(bool visible)
{
    if (visible == m_visible)
        return;

    if (visible)
        XMapWindow(m_display, m_window);
    else
        XUnmapWindow(m_display, m_window);

    m_visible = visible;
}
```
