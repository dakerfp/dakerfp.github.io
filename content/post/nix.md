+++
title = "Nix"
date = "2013-06-11"
+++

For the last few months my team (OpenBossa) at INDT (Instituto Nokia de
Tecnologia) have been working on [WebKitNix](http://nix.openbossa.org)
(Nix for short). WebKitNix is a new WebKit2 port, fully based on POSIX
and OpenGL. We also use CMake build system (like GTK and Efl), GLib,
libsoup (networking) and Cairo (2D graphics) as dependencies. It also
uses Coordinated Graphics and Tiled Backing Store from WebKit2. Many of
its building blocks are shared with others ports already on trunk.

What is it?
-----------

The Nix port offers a C API for rendering a WebView within a OpenGL
context, based on WebKit2 API. You can use Nix to create applications
such as a web browser or a web runtime. WebKit2 run the context for each
web page in a different process. This process isolation keeps the UI
responsive, with smooth animations and scrolling, because it does not
get blocked by Javascript execution.

We want to ease the work of Consumer Electronics developers who wants to
have a web runtime in their platform, without the need to create another
WebKit port. That is why Nix has less dependencies than Efl, GTK or Qt,
which should also be ported to the target platform.

Nix API also enables the application to interact with the Javascript
context. So it is possible to add new APIs, to handle application
specific features.

![](/img/nix.png)


How did it started?
-------------------

The OpenBossa team used to maintain the Qt WebKit port for years,
helping Nokia/Trolltech. But then, in the last years, from the
experience gathered with the [Snowshoe](http://snowshoe.openbossa.org/)
browser, handling with dependencies (such as QtNetwork) that were much
bigger than we really needed. We tried to replace some dependencies of
QtWebKit and later Efl to see how minimal WebKit could be. So we took
some steps:

1.  Initial idea: platform/posix or platform/glib (share code)
2.  Ivolved problem: we wanted to have different behaviors for
    QQuickWebView -\> Qt Raw WebView
3.  Network: QtWebKit + Soup experiment
4.  Efl Raw WebView experiment
5.  Efl Without Efl :-)
6.  Nix

How to use it?
--------------

When you compile [Nix source code](https://github.com/WebKitNix/webkitnix)
you can run the MiniBrowser to test it:

`$ $WEBKITOUTPUTDIR/Release/bin/MiniBrowser http://maps.nokia.com`

![](/img/nix-map.png)

[MiniBrowser code](https://github.com/WebKitNix/webkitnix/tree/master/Tools/MiniBrowser/nix)

The Nix-Demos repository offers some example code, including a Glut
based browser and minimal Python bindings for Nix:
[https://github.com/WebKitNix/nix-demos](https://github.com/WebKitNix/nix-demos).

On Nix-Demos we also have a Nix view, using only a DispmanX and OpenGL
ES 2 working on the Raspberry Pi. To compile this demo, you will need
our [RaspberryPi SDK](https://github.com/WebKitNix/nix-rpi-sdk).

![](/img/nix-rpi2.jpg)

There is even a browser with its UI written in HTML:
[Drowser](https://github.com/WebKitNix/drowser)

Feel free to contact us on `#webkitnix` at freenode.

Roadmap
-------

Our plan is to upstream Nix in WebKit trunk by June/2013. Then, keep the
maintainence and focus on the web platform, including some new HTML5
features, such as WebRTC.
