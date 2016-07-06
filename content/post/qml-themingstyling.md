+++
title = "QML Theming/Styling"
date = "2012-04-15"
+++

Intro
-----

As I mentioned in my last post, I and my team in INdT are researching
on how to improve the Styling  new way of creating styled QML widgets
components (Button, Sliders, ComboBox, etc.). We've been involved for
such a long time in the development of Qt Components for the Harmattan,
and me personally worked on the Plasma Components widget set.

We also did a some big desktop and mobile applications (such as
[Snowshoe](http://snowshoe.qtlabs.org.br/) and
[PagSeguro NFC Payment](http://conversations.nokia.com/2012/04/13/making-mobile-payments-in-brazil-with-nfc/))
which required us to create lots of custom widgets. Obviously there's a
lot of redundant code between the applications that could belong to a
common code base.

Besides the existence of the [Qt Components](https://qt.gitorious.org/qt-components/),
the styling of widgets is an issue it doesn't solves as well. Nowadays if you want a
new style, you'll have to create a set of QML components from scratch or
to base on an existing widget set, which would increase the development
and code fragmentation. One way to style your widgets is to rely on the
old and confusing QStyle classes, and there is no easy way to it purely
in QML.


Issues and Requirements
-----------------------

Recently, lots of discussions about a new styles API happened on the Qt
development mailing list. Some of the issues as we considered were
summarized by Alan Alpert in a
[thread](http://qt-project.org/forums/viewthread/16341) on Qt Project's
forum. Here are some of the points he collected from the discussion in
the mailing list:

- One source, multiple target platforms
- Change the appearance of standard controls in my application
- Style QWidget and QQuickItem UIs with the same code
- Inside QML items, draw native controls without using QML

In addition to those concerns, based on our experience on desktop and
mobile applications we have a few other important issues to guide us:

-   Configure current theme/style from the QML
-   Use QML for configure new Styles for widgets
-   Minimize code size for application developers
-   Wisely use SceneGraph to improve performance
-   Don't need to link with the old QtWidgets module

The current implementation of the custom components in the [Desktop
Components](https://qt.gitorious.org/qt-components/desktop) solves a few
of these problems. However one of the biggest problem is depending on
QtWidgets, not having an easy style change support and not properly
using the SceneGraph for painting the widgets.


QtQuickStyles
-------------

To research how to solve these issues we've decided to create the
QtQuickStyles module. The general idea for this module is to provide a
simple way of styling QML widgets (think of Qt Components). The current
research is about creating an API to allow proper platform look and feel
integration without depending on QtWidgets, and easily theme your whole
application with your custom style, if you want another theme besides
the underneath platform look and feel. We want that the developers be
able to do it all purely in QML. All the collected issues are being
considered for the API.

The current work is available here:

`git clone git://code.openbossa.org/projects/qtquickstyles.git`

Now, our focus is to adapt the existing code base, from QStyle, for
painting elements in QML as a short term target so we can have working
code faster, and supporting all the same platforms and styles that
QtWidgets does.


QStyle
------

Today, on Qt code, each style knows how to paint all the control
elements, and they all follow an interface (defined by QStyle class). If
you want a new style you will have to implement a new QStyle subclass.
The following scheme may clear up your mind:

![Current QStyle architecture](/img/qstyles.png)

Additionally, if you take a look at the QStyle code, you can see a
bunch of mixed code, taking care of drawing all type of widgets. Almost
all the methods have a huge switch case, each taking care of an specific
action of an specific widget. When someone put the eyes on it for the
first time, thinks it is too complicated to understand and very messy
(and indeed it is). Additionally, a code like this leads to a huge
complexity to maintain and understand it.

So, why can’t each widget take care of its own features and painting?
Why we could not have an interface that would define how a button is
painted, for instance? Doing it this way, if we have some work to be
done on the button (again the button example), it would only be done on
this button class. Furthermore, if we have modifications on how the
button with the plastique style, for instance, is painted, we have only
to do it on the plastique button class. This approach leads to more
understanding on the overall code and also makes it easy to maintain,
isolating the problems that can appear, to the widget itself.

To do this, a component must be a control or composed of sub controls
(such as the Slider's groove, handle and tickmarks) and for each control
it'll use a QQuickItem that reads the widget's state and "paints" itself
with as it is the style.


QtQuickStyles in use
--------------------

We've done a few brainstorm for defining the current API to use the
style in the components. The following code is the example of how
our API was thought to be used:

```qml
// Regular Button having the underneath platform look and feel
import QtQuick 2.0
import QtUiComponents 1.0
Button {
    width: 140
    height: 40
    text: "Native Button"
}
```

We are going to explain later how the style resolves the current
platform being used.

Override platforms style:

```qml
// Regular Button having the plastique look and feel
import QtQuick 2.0
import QtUiComponents 1.0
import QtUiStyles 1.0
Button {
    width: 140
    height: 40
    text: "Plastique Button"
    // This use case is not implemented exported yet, but it will be
    style: PlastiqueButtonStyle { }
}
```
Using a user defined style:

```qml
// Regular Button
import QtQuick 2.0
import QtUiComponents 1.0
Button {
    width: 140
    height: 40
    text: "User-defined Button"
    // Accepts a QML Component
    style: Rectangle {
        color: "gray"
        Text {
            anchors.centerIn: parent
            text: button.text // button reference is injected when the internal Loader loads it
        }
    }
}
```
 
We can see, by the examples above that we can easily uses a Button
component and gives it the look and feel that we want, by only playing
around with the style property. In order to follow a well discussed and
already implemented Component's API, we're using the
[Qt Components Common API](https://bugreports.qt.nokia.com//browse/QTCOMPONENTS-200)
specification for creating the components in the repository.


Custom Styles
-------------

To solve the issue of being easy to create a new style, we are also
adding component helpers that exports properties that commonly changes
from one application to another. These configurable properties were
selected based on our experience of creating QML components.

The custom style API defined for the Button and Style can be found
[here](http://wiki.qtlabs.org.br/projects/qtquickstyles/customstyleapi).

```qml
// Regular Button
import QtQuick 2.0
import QtUiComponents 1.0
import QtUiStyles 1.0
Button {
    width: 140
    height: 40
    text: "Custom Style Button"
    style: CustomButtonStyle {
        background: "myButton.png"
        pressedBackground: "myPressedButton.png"
    }
}
```


ApplicationStyle API
--------------------

As listed in the issues list, it's interesting that we had an easy way
of setting the application theme/style in QML. We came to an API that
let the developer change the style of all the components in the
application that uses the default style. The example below shows how to
play around with the application theme/style:

```qml
import QtQuick 2.0
import QtUiStyles 1.0
Item {
    ApplicationStyle {
        id: appStyle
        buttonStyle: CustomButtonStyle {
            background: "myButton.png"
            pressedBackground: "myPressedButton.png"
        }
    }

    Button {
        text: "Button with custom Style"
    }

    Button {
        text: "Button with plastique Style"
        // Theme changes doesn't affect when style property is set
        style: PlastiqueButtonStyle { }
    }

    Button {
        text: "Button with appStyle"
        style: appStyle.buttonStyle // Explicitly binds button style with a style
    }
}
```


Implementation Details
----------------------

We have a global class, called QUiStyle, which (practically) has
getters for each widget style who defines how it is painted. So, we
would have some methods like: buttonStyle(), checkBoxStyle(),
radioButtonStyle() and so on. Each specific style, e.g. Plastique,
Cleanlooks, Widows, etc, will inherits from this base class and set the
members that will be returned by the getters, to its specific widget
style class, for instance, on the QUiPlastiqueStyle class, we will set
buttonStyle member to an instance of QUiPlastiqueButton. So, it is easy
for a developer to add its own Style (if he desires to make it on C++),
he only has to define his new style class, inheriting from QUiStyle, and
set the style widget members of QUiStyle to its own style widgets.

The following diagram can ease your understanding of the current code
structure:

![Proposal for a new QStyle modular architecture](/img/quistyles.png)

For each type of widget we will have a class that calls the correct
style class to paint it. For example, for the button, we have a class
called QUiButtonStyle, which will ask for the QUiStyle a reference for
the platform button style (calling the buttonStyle() method). Then, it
will be drawn with the current platform look and feel. All the other
widgets follows the same workflow.

![Current implementation for the style components](/img/quicomponentstyle.png)

In order to theme your application, we have a global object called
appStyle and a QML component, called ApplicationStyle (presented in the
QML example above). The ApplicationStyle component as a style property
for each type of widget, e.g. buttonStyle, checkBoxStyle,
radioButtonStyle, etc. These properties binds to the appStyle object
analogue properties. So, if you want all your buttons of your
application to have a CustomButtonStyle, you can simply do it as the
example.


Development Status
------------------

Currently, we are focusing on creating a good set of working widgets
for the Plastique style, but we are planning to implement another style
parallelly, the CleanLooks is the favourite candidate.

Now, if you take a look at our repository, you will find the following
widgets:

- Button
- CheckBox
- RadioButton
- Slider

Besides this, we have a CustomStyle for each one, e.g.
CustomButtonStyle, CustomCheckBoxStyle, etc. If a user does not want to
use the platform look and feel or create a style from scratch, he can
use this Custom API to easily create his own styles.

The next widget that is planned to be developed is the ComboBox, also
with his Custom Style.


Remarks
-------

This first release was done using the QPainter to paint the widgets,
instead of using the new SceneGraph. This was done in order to speed up
our development and have a nice set of widgets working as fast as
possible. In the future (close future) we will get rid of all QPainter
based classes and replace them to use SceneGraph. The current custom
styles already does that, but we would like to have them working for the
platform styles as well, and we are already researching it.

The code in repository is mess because we are still doing a lot of
changes, but we need to make it make it more stable and follow the Qt
addons repository architecture until the end of the month. If you have
any opinion, doubt or suggestion, please let us know.
