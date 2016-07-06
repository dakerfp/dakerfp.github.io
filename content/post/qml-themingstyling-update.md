+++
title = "QML Theming/Styling (Update)"
date = "2012-05-14"
tags = ["kde", "qml", "qt", "qt5", "qtcomponents", "qtquick", "qtquickstyles", "styles", "theme"]
+++

This post is an update about the research project from my team,
described [a few weeks ago](http://codecereal.blogspot.com.br/2012/04/qml-themingstyling.html).

From the time we published the last post about QML Styling until now we
have worked on this set of issues/features:

- Get feedback about research project
- Combo Box Component
- Combo Box Customizable Style
- Combo Box Plastique Style
- SubControl Styling
- Understand SceneGraph internals
- Understand other native platform internals

I will detail what was possible to make for each of these topics in
sessions below.

What is our vision now?
-----------------------

Last week, we have read a few blog posts, and talked with a few Qt & KDE
application developers about what should be the priorities for creating
desktop and mobile applications. I have presented our proposed solution
for using native look and feel for QML widgets, how to create custom
styles from scratch, using the CustomStyles helper, and how to apply
them with the ApplicationStyle API.

Based on the feedback and the blog posts, my team sat down and came with
the following set of statements which summarize our vision for what
sould be our focus of our current research:

1. Usable QML components with native styles working ASAP

    Developers want to code entire application UI with QML having native
    look and feel.

2. Easy customization

    It's all about making easier to create components with different look
    only by filling in some templates to avoid code repetition for standard.
    These custom styles are targeted to be like a short cut, obviously for
    more complex behaviour, you will need to create your own style.

3. Powerful customization

    Enabling to use QtQuick components as the style can make widgets look
    fluid. It's desirable that the new styling mechanism is at least as
    powerful as QStyle is today. As a first shot we want to enable styling
    do at least what QtWidgets style does. The main point here is to
    maximize the results and minimize ramblings about what is style or not.

4. Styling modularization

    By spliting the old style scheme in a set of widget style, enables us to
    create the style for each component/platform independently instead of
    the monolithic QStyle. Now it's easier to mix styles and change them on
    demand more easily.

5. Disruption with QtWidgets

    We wish to make this component set free from QtWidgets modules. One of
    the reasons is because now it is considered
    [done](http://labs.qt.nokia.com/2012/04/18/qt-5-c-and-qt-widgets/) and
    it's desirable for the new components set that it can be expanded. We
    also don't want to link with QtWidgets module, because the real
    dependency should be the QStyle only. The current
    [ApplicationStyle](http://codecereal.blogspot.com.br/2012/04/qml-themingstyling.html)
    approach, shows us that the styles depends only on QtQuick. One of the
    possible paths to achive this is:

    1.  Move QStyles out of QtWidgets with a few adaptions on it.
    2.  Create a SceneGraph based native styles when possible

Combo Box
---------

We decided to choose the ComboBox component to work on because it is one
of the most complex (if it isn't the most). Because of the complexity,
we hoped that during its development we could be enlightened of knowing
if we are in a correct path, what still misses, and what should be the
next steps.

As we did in the Slider approach, which was divided in 3 different
[subcomponents](http://codecereal.blogspot.com.br/2012/04/qml-themingstyling.html):

- Handle
- Groove
- Tickmarks

While creating the ComboBox, we decided to divide it in 4 other subcomponents:

![](/img/combo-th.png)


- ArrowStyle
- BackgroundStyle
- TextEditStyle
- DropListStyle

We basically mimicked how QStyle splits the QComboBox painting into
subcontrols. The drop list was also delegated a sub style as QComboBox
does with it's internal QListView. We haven't worked on the drop list
style since it would require a native style such as Plasma's
ListItemView, which also would rely on a ScrollBar.

Creating the combo box component showed us that positioning and size
hints can be more tricky than it looks like.

The ComboBox got stuck in a few parts and unfortunately it's not
complete right now. However we took the questions and answers from its
development. :-/

Positioning and Size Hints
--------------------------

This topic of discussion came out when we were thinking about a
theoretical style in which the ComboBox would be in the left. One of the
issues we had in mind while developing the editable ComboBox was how to
set a MouseArea that can know when set the focus to the text edit or to
open the drop list. This would be possible to be done with current
QStyle, since on it's approach the QWidget reads the subcomponent's size
hints by the `subControlRect` method from QStyle.

We would like to have this positioning information on the style as well.
The approach can be similar to what happens with the size, which you can
read it from the widget reference.

The following piece of code is a simple example of how size hints can be
taken:

```qml

// ComboBox.qml
Item {
    property alias arrowStyle: arrowControl.sourceComponent

    Loader {
        id: arrowControl
        width: arrowControl.implicitWidth
        height: arrowControl.implicitHeight
    }

    MouseArea {
        anchors.fill: arrowControl
        onClicked: {
            // do some action
            // ...
        }
    }
}
```

ArrowStyle defines the implicit size, which works as a size hint, and
the position where they are. These properties together can work analogue
to `subControlRect`, as they hold the same info. The component may
ignore such hints and override the properties values, such as Slider's
Handle style position.

```qml
// MyComboBoxArrowStyle.qml
Image {
    implicitWidth: 50
    implicitHeight: comboBox.height
    x: comboBox.width - width // Arrow could also appear on the left by setting x = 0
    source: "arrow.png"
}
```

One may ask "Can't I have a round button with a circular hit area?"
That's more complex than just setting hints for the geometry of sub
control styles. As we defined in our view we're trying to be at least as
powerful as QStyle. We consider that, by now, we should be strict at
least about the interaction styling of the components themselves. From
my point behaviour difference should be defined as the component API.

### Sub StyleComponents Sets

Another discussed topic was about the fragmentation of the style
property of the components. For instance, take the following Slider
style code:

```qml
// Slider style now
Slider {
    grooveStyle: CustomGrooveStyle { ... }
    handleStyle: CustomHandleStyle { ... }
}
```

The Slider style property is fragmented as more than one property. We
thought that these properties could be centralized with a SliderStyle as
an aggregator object. This helps API clarity for style manipulation
since we can play with a single object reference that represents the
component style, enabling to handle it atomically.

```qml
// Proposed Slider style usage
Slider {
    sliderStyle: CustomSliderStyle { ... }
}
```

with CustomSliderStyle as:

```qml
// Proposed Slider style creation
// CustomSliderStyle.qml

// Aggregated style object
SliderStyle {
    grooveStyle: CustomGrooveStyle { ... }
    handleStyle: CustomHandleStyle { ... }
    tickmarksStyle: CustomTickmarksStyle { ... }
}
```

or more compactly:

```qml
Slider {
    sliderStyle: SliderStyle {
        grooveStyle: NativeGrooveStyle { ... }
        handleStyle: CustomHandleStyle { ... }
    }
}
```

or even:

```qml
Slider {
    sliderStyle {
        grooveStyle: NativeGrooveStyle { ... }
        handleStyle: CustomHandleStyle { ... }
    }
}
```

This issue is only an idea only discussed between ourselves. It would be
nice to have feedback about these API.

Insights from SceneGraph & QStyle study
---------------------------------------

The isolated study of the scene graph internals (getting rid of
QQuickPaintedItem), and how it could be used to create the new styles
directly on it, didn't told us much in fact. Only that is better we keep
doing these styles in QML and using Scene Graph itself to create sub
elements that needs a more refined handling.

On the other hand, the Windows and Mac styles investigation was very
important to decide our next steps. It showed us that these styles uses
platform native APIs to draw the native widgets on each platform on
pixmaps. So we would have to deeply study these API to create our own
implementation of native styles using the scene graph. For these reasons
isn't too simple to give up from QQuickPaintedItem some time to going
deep on them right now since our time and head count is limited.

Two steps forward, one step back
--------------------------------

After the feedback from other developers, one of the main thing people
want more is to have a widget set working with the native look and feel
as soon as possible. Keeping this as our primary focus, we will left the
restriction of depending on QtWidgets for now. So we will focus on
having a working solution that can be easily replaced after.
Fortunately, our proposed modular solution for styling fills that
requisite.

Labels: [kde](http://codecereal.blogspot.com.br/search/label/kde),
[qml](http://codecereal.blogspot.com.br/search/label/qml),
[qt](http://codecereal.blogspot.com.br/search/label/qt),
[qt5](http://codecereal.blogspot.com.br/search/label/qt5),
[qtcomponents](http://codecereal.blogspot.com.br/search/label/qtcomponents),
[qtquick](http://codecereal.blogspot.com.br/search/label/qtquick),
[qtquickstyles](http://codecereal.blogspot.com.br/search/label/qtquickstyles),
[styles](http://codecereal.blogspot.com.br/search/label/styles),
[theme](http://codecereal.blogspot.com.br/search/label/theme)

