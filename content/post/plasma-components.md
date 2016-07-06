+++
title = "Plasma Components"
date = "2011-05-19"
tags = ["gsoc", "kde", "plasma", "qtcomponents", "qtquick"]
+++

Hi,
 
I will talk about my GSoC project (\\m/), the plasma components.
As you may know, QML is a declarative language to build rich interfaces
introduced in Qt 4.7 by providing simple primitives. As it is a powerful
way to develop interfaces and it's the future of UI development for Qt
was necessary to make the plasma support it.

Create interfaces in QML is really easy and fast but sometimes we need
common widgets and may be boring to reimplement and replicate them in
every application we create (e.g. Button, Slider, ScrollBar). For avoid
this code replication, the Qt Components project was created to unify an
API for a set of components.
 
The current (often updated) defined components and it's API can be
found at
[QTCOMPONENTS-200](http://bugreports.qt.nokia.com/browse/QTCOMPONENTS-200).
The Qt Components intends to be a cross platform, but sometimes we need
to have a closer integration of the component and the platform. In
plasma desktop we want to show tooltips or use the theme svg images for
example. The common API defines a set of properties a component must
have, but doesn't disallow to have extra properties and functionalities
(suggestions ?). Creating the plasma components with the theme
integration and adding plasma behaviour is what my GSoC project is about
:-).

I already started working on the plasma components. They can be found
in the kde-runtime repository at the plasma/declarative branch. And more
speciffically in the plasma/declarativeimports/plasmacomponents
directory. The components which already there are not yet done, they
lack of tooltips, keyboard events handling, focus policy, and some
(CheckBox and RadioButton) hasn't images yet. However they're fully
functional and they cover all the properties and behaviours defined in
the common API. Here is the list of the components in this state
mentioned above (until this post publication):

-   BusyIndicator
-   Button
-   CheckBox
-   RadioButton
-   Slider

These are components not present in the common API but they're highly
wanted in plasma:

-   ScrollBar
-   ListItem
-   ListItemView
-   ListHighlight

To test these components, its also kept a components gallery
(plasma/declarativeimports/test/Gallery.qml) that you may view it with
qmlviewer (after installing the components).

![Gallery screenshot](/img/gallery.png)
  
Feedback is highly appreciated. ;-)

