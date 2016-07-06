+++
title = "Back to Plasma Components"
date = "2011-07-18"
tags = ["gsoc", "kde", "plasma", "qt", "qtcomponents"]
+++
This last month I've been working slowly on my GSoC project due to the
university activities (due to Brazilian academic calendar). And my
thesis, which I shall talk in other post. But at least now I'm
undergraduated!

As I explained in my last post about [Plasma Components],
my GSoC project. I'm building graphics components for developers to
build plasmoids in QML using non trivial common components such as
Sliders, ScrollBars and RadioButtons. After the break I've done these
components following the [Qt Components common
API](http://bugreports.qt.nokia.com/browse/QTCOMPONENTS-200):

-   Switch
-   ButtonRow
-   ButtonColumn
-   ScrollBar
-   ScrollDecorator

Keyboard event handling and focus policy for the new and old components
were added in this sprint. I also spent a lot of time refactoring some
components. I think their code is much better now.

I'm also adding more complex use cases at the components gallery (at
kde-runtime/plasma/declarativeimports/test/gallery). By the way, this is
screenshot of the new gallery:

![](/img/gsoc-gallery.png]

Plasma Components Gallery
-------------------------

Part of the work was just straightforward, but there are some doubts I
would like to ask which you think is the best, because some of the
components behaviour are not defined.

1.  Should ScrollDecorator appear only when it's flickableItem is
    flicked, like in the mobile use case?
2.  ScrollDecorator must should look like ScrollBar or have it's own
    appearence?
3.  There are no SVG graphics for CheckBox, RadioButton and Switch.
    Currently there are placeholders. What can I do?
4.  Currently, when you click a component, it gains the focus. This
    logic must be in the components as it is? Or should left it external
    to the button.
5.  The Qt Components doesn't define any enabled property, for any
    components. I think it's important to have such a property in all
    interactive components. What do you think about it?

Any suggestion is highly appreciated.

I expect to give other update as soon as I have something more to
report.

