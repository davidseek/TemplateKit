# TemplateKit
[![Swift](https://img.shields.io/badge/swift-3-orange.svg?style=flat)](#)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![CocoaPods Compatible](https://img.shields.io/cocoapods/v/TemplateKit.svg)](https://img.shields.io/cocoapods/v/TemplateKit)
[![Platform](https://img.shields.io/cocoapods/p/TemplateKit.svg?style=flat)](http://cocoadocs.org/docsets/TemplateKit)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://opensource.org/licenses/MIT)

[React](http://facebook.github.io/react/)-inspired framework for building component-based user interfaces in Swift.

|      | Features                                   |
| ---- | ---------------------------------------- |
| 🐤   | Completely native - write your app in Swift |
| 📃   | Declarative - define your UI using markup |
| 📦   | Components - encapsulate functionality into reusable chunks |
| 📐   | Layout - Flexbox for layout, just like on the web |
| 🖋   | State - automatically flush state changes to UI |
| ⚖    | Diffing - the minimum set of updates are flushed to UI |
| 🚀   | Performance - diffing and layout are done on background threads |
| ⏲   | Animation - built-in support for animating any property |

## Example
You define your UI using a simple markup language, which is inspired by HTML and CSS. This UI definition is rendered into a tree of native elements.

#### Component.xml
```html
<template>
  <style>
    #container > .button {
      color: #000;
    }
    
    #container > .button-selected {
      color: #f00;
    }
  </style>

  <box id="container">
    <text text="$properties.title" />
    <text text="Click me!" onTap="handleClick" classNames="$textClasses" />
  </box>
</template>
```
#### Component.swift
Functionality and state is encapsulated into components, which do things like handle user events and flush state changes to the UI. Components have strongly typed `State` and `Properties` values, that are used to figure out what ends up getting pushed out to UIKit.
```swift
struct ComponentState: State {
  var selected: Bool?
}

struct ComponentProperties: Properties {
  var core = CoreProperties()
  var title: String? = "This is a default title"
}

class MyComponent: CompositeComponent<ComponentState, ComponentProperties, UIView> {
  // Stored properties on the component are made available to template.
  var textClasses: String?
 
  // As are functions, referenced by their selector name.
  @objc func handleClick() {
    updateComponentState { state in
      state.selected = !state.selected
    }
  }
 
  override func render() -> Element {
    textClasses = state.selected ? "button" : "button-selected"
  
    return render("http://localhost:8000/Component.xml")
  }
}
```
#### ViewController.swift
Rendering components is as easy as calling a `render` function, which asynchronously computes and flushes a component to the supplied container view.
```swift
override func viewDidLoad() {
  super.viewDidLoad()
 
  UIKitRenderer.render(component(MyComponent.self), container: self.view, context: self) { component in
    self.component = component
  }
}
```

See the included [Example](https://github.com/mcudich/TemplateKit/tree/master/Example) project for more examples of how to use TemplateKit.

## Why?

#### Swift
Because you like writing your apps completely in Swift. TemplateKit is fully native and compiled.

#### Declarative Style
Writing user interfaces in a declarative style makes it easier to reason about how model data and user actions affect what gets rendered. Out-of-the-box support for XML. Extensible if you want to add your own template format (e.g., protocol buffers).

#### Components
Components make it easy to encapsulate application functionality into re-usable building blocks. These blocks can then be composed to create more complex interfaces.

#### Layout
Flexbox-based layout primitives allow developers to use the the same expressive layout system available in modern browsers.

#### Asynchronous Rendering & Performance
All layout computation, text sizing, tree diffing, image decoding is performed in the background. This keeps the main thread available for responding to user actions. Only the absolute minimum set of changes needed to update the view hierarchy are actually flushed to the rendered views.

#### CSS
Use stylesheets to style components, just like you do on the web.

#### Animations
Animate layout, style and arbitrary properties using an intuitive API. 

#### Live Reloading
Automatically reload changes to user interfaces without having to re-build binaries or restart your application. Usable in both development and production environments.

#### Extensible
Add custom components, custom native views, custom template loading schemes and more.

#### Easy to try
Plug it in anywhere you want to render a view in your application. Plays nicely with the rest of your app.

## Installation

#### Carthage

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

Add the following line to your [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile):

```ogdl
github "mcudich/TemplateKit"
```

Run `carthage update`, then make sure to add `TemplateKit.framework`, `CSSLayout.framework`, and `CSSParser.framework` to "Linked Frameworks and Libraries" and "copy-frameworks" Build Phases.

#### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate TemplateKit into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target '<Your Target Name>' do
    pod 'TemplateKit', '~> 0.1.0'
end
```

Then, run the following command:

```bash
$ pod install
```

## Requirements

- iOS 9.3+
- Xcode 8.0+
- Swift 3.0+

## Communication

- If you **found a bug**, open an issue.
- If you **have a feature request**, open an issue.
- If you **want to contribute**, submit a pull request.

## How does it work?
At its core, TemplateKit is comprised of `Element` and `Node` instances. Elements are used to describe trees of nodes, which can be anything that implements the `Node` interface. Nodes are used to vend out and manage view hierarchies.

Out of the box, there are several `Node` implementations that make it easy to set up UI hierarchies: `Component`, `ViewNode`, and a set of native controls like buttons, text labels, text fields and so on.

Building a component is as simple as subclassing `Component`, overriding its `render()` function, and deciding the set of properties it might accept and use as part of rendering. `render()` simply needs to return a `Template`, which can be constructed programmatically, or via an XML document (or other custom payload). When it comes time to render your component into a view, you simply call `UIKitRenderer.render`, and pass in the view that should contain your component's rendered output. This will in turn call `render()` on your component instance, compute the layout and styles for the view tree, build this tree and then apply the layout and styles to it as appropriate.

When it comes time to update your component's state, you can call `updateState` from within your component implementation. This function receives a function that is passed the current state value (each `Component` can declare a `State` type, in the same way it declares a `Properties` type). This function in turn enqueues an update to the component, which will cause it to re-render, taking into account whatever changes were made to the state. This update is intelligent, and compares the current incarnation of the rendered view tree against the proposed element tree. Only the deltas between these two are flushed out to the view layer.

## Opaque Views
If there are parts of your UI that are easier to deal with as plain `UIViews`, TemplateKit provides a simple abstraction `Node` called `ViewNode` that allows you to include these "opaque" views as part of any TemplateKit-managed tree. TemplateKit stays out of the way, and simply sets the `frame` of these views for you, so they sit nicely within in whatever UI tree you've composed.

## Collections
TemplateKit provides `UITableView` and `UICollectionView` subclasses which are able to load, and asynchronously size and render `Components` into cells with just a little bit of configuration. Tables and collections can be used via `Table` and `Collection` components, or simply wrapped as `ViewNode` instances. The `Table` and `Collection` components have built-in support for diffing, so that data-source updates result in the minimum set of operations required to have the respective UIKit views reflect data changes. See [Diff.swift](https://github.com/mcudich/TemplateKit/blob/master/Source/Utilities/Diff.swift) for more information.

## How's this different from React Native?
TemplateKit is implemented in Swift (and a bit of C). If you like writing entirely in Swift, then this framework might be for you.

React Native relies on a very well-tested library (React), and has been shipping in popular apps for some time now. This means it probably has way fewer rough edges, has sorted out many performance issues TemplateKit has yet to face, and so on.

## What's Missing
A lot.

There's no AppKit support yet (though it would be straightforward to add). Lots of tests have yet to be written. Performance testing has yet to be done. The entirety of the applicable CSS spec is not supported. Animation features are rudimentary at best. Many gesture types need to be added. And much more.

If you'd like something added, please file a feature request or send a pull request!

## Inspiration
- [React](https://github.com/facebook/react)
- [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)

## See Also
If TemplateKit isn't exactly what you're looking for, check out these other great projects!
- [Few.swift](https://github.com/joshaber/Few.swift)
- [Render](https://github.com/alexdrone/Render)
- [React Native](https://facebook.github.io/react-native/)
- [LayoutKit](https://github.com/linkedin/LayoutKit)
- [ComponentKit](https://github.com/facebook/componentkit)
- [HubFramework](https://github.com/spotify/HubFramework)
