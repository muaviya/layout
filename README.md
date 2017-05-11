# Layout

- [Introduction](#introduction)
	- [What?](#what)
	- [Why?](#why)
	- [How?](#how)
- [Usage?](#usage)
    - [Installation](#installation)
    - [Integration](#integration)
    - [Live Reloading](#live-reloading)
    - [Constants](#constants)
    - [State](#state)
    - [Actions](#actions)
    - [Outlets](#outlets)
    - [Delegates](#delegates)
    - [Composition](#composition)
- [Expressions](#expressions)
	- [Layout Properties](#layout-properties)
	- [Strings](#strings)
	- [Colors](#colors)
	- [Images](#images)
	- [Fonts](#fonts)
	- [Attributed Strings](#attributed-strings)
- [Advanced Topics](#advanced-topics)
	- [Manual Integration](#manual-integration)
	- [Custom Components](#custom-components)
- [Example Projects](#example-projects)
	- [SampleApp](#sampleapp)
	- [UIDesigner](#uidesigner)

# Introduction

## What?

Layout is a framework for implementing iOS user interfaces using runtime-evaluated expressions for layout and (optionally) XML template files. It is intended as a more-or-less drop-in replacement for Storyboards, but offers a number of advantages.

The Layout framework is *extremely* beta, so expect rough edges and breaking changes.

## Why?

Layout seeks to address a number of issues that make StoryBoards unsuitable for large, collaborative projects, including:

* Proprietary, undocumented format
* Poor composability and reusability
* Difficult to apply common style elements and metric values without copy-and-paste
* Hard for humans to read, and consequently hard to resolve merge conflicts
* Limited WYSIWYG capabilities

Layout also includes a replacement for AutoLayout that aims to be:

* Simpler to use for simple layouts
* More intuitive and readable for complex layouts
* More deterministic and simpler to debug
* More performant (at least in theory :-)

## How?

Layout introduces a new node hierarchy for managing views, similar to the "virtual DOM" used by React Native.

Unlike UIViews (which use NSCoding for serialization), this hierarchy can be deserialized from a lightweight, human-readable XML format, and also offers a concise API for programatically generating view layouts in code when you don't want to use a separate resource file.

View properties are specified using *expressions*, which are simple, pure functions stored as strings and evaluated at runtime. Now, I know what you're thinking - stringly typed code is horrible! - but Layout's expressions are strongly-typed, and designed to fail early, with detailed error messages to help you debug.

Layout is designed to work *alongside* ordinary UIKit components, not to replace or reinvent them. Layout-based views can easily be embedded inside storyboards, and nib-based views can be embedded inside Layout-based views and view controllers.


# Usage

## Installation

To install Layout using CocoaPods, add the following to your Podfile:

	pod 'Layout', :git => 'git@github.schibsted.io:Rocket/layout.git'

This will include the Layout framework itself, and the open source Expression library, which is the only dependency.

## Integration

The core API exposed by Layout is the `LayoutNode` class. Create a layout node as follows:

    let node = LayoutNode(
    	view: UIView(),
    	expressions: [
    		"width": "100%",
    		"height": "100%",
    		"backgroundColor": "#fff",
    	],
    	children: [
    		LayoutNode(
    			view: UILabel(),
    			expressions: [
    				"width": "100%",
    				"top": "50% - height / 2",
    				"textAlignment": "center",
    				"font": "Courier bold 30",
    				"text": "Hello World",	
    			]
    		)
    	]
    )
    
This example code creates a centered `UILabel` inside a `UIView` with a white background that will stretch to fill its superview once mounted.

To mount a `LayoutNode` inside a view or view controller, the Layout framework provides a number of integration solutions from high-level to low-level. The simplest, most high-level approach is to subclass `LayoutViewController` and use one of the following three approaches to load your layout:

    class MyViewController: LayoutViewController {
    
        public override func viewDidLoad() {
        	super.viewDidLoad()
        	
        	// Option 1 - create a layout programmatically
        	self.layoutNode = LayoutNode( ... )
        	
        	// Option 2 - load a layout synchronously from a bundled XML file name
        	self.loadLayout(named: ... )
        	
        	// Option 3 - load a layout asynchronously from an XML file URL
        	self.loadLayout(withContentsOfURL: ... )
        }
    }
    
For simple layouts, option 1 is a convenient solution that avoids the need for an external file. For more complex layouts, option 2 is recommended, as it allows for *live reloading*, which can significantly reduce development time. To use option 2, the file must be located inside the application resource bundle.

Option 3 can be used to load a layout from an arbitrary URL, which can be either a local file or remotely hosted. This is useful if you need to develop directly on a device, because you can host the layout file on your Mac and connect to it from the device. It's also potentially useful in production for hosting layouts in a CMS system, however note that `loadLayout(withContentsOfURL:)` offers limited control over caching, etc. so for production purposes it may be better to download the XML template to a local cache location first and then load it from there.

## Live Reloading

The `LayoutViewController` provides a number of helpful features to improve your development productivity, most notably the *red box* debugger and the *live reloading* option.

When running in DEBUG mode, if the Layout framework throws an error during XML parsing, node mounting or updating, the `LayoutViewController` will detect it and display the *red box*, which is a full-screen overlay displaying the error message along with a reload button. Pressing reload will reset the layout state and re-load the layout.

When running in the simulator, if you press the reload button, `LayoutViewController` will attempt to find the original source XML file for the layout and reload that instead of the static version bundled into the compiled app. This means that you can go ahead and fix the error in your XML file, then reload it *without restarting the simulator, or recompiling the app*.

You can reload at any time, even if there was no error, by pressing Cmd-R in the simulator (not in Xcode itself). `LayoutViewController` will detect that key combination and reload the XML, provided that it is the current first responder on screen.

**Note:** This only works for changes you make to the layout XML file, not to Swift code changes in your view controller, or other resources such as images.

This live reloading feature, combined with the gracious handling of errors, means that it should be possible to do most of your interface development without needing to recompile the app. This can be a significant productivity boost.

## Constants

Static XML is all very well, but in the real world, app content is dynamic. Strings, images, and even layouts themselves need to change dynamically based on user generated content, locale, etc.

`LayoutNode` provides two mechanisms for passing dynamic data, which can then be referenced inside your layout expressions, *constants* and *state*.

Constants, as the name implies, are values that remain constant for the lifetime of the `LayoutNode`. The constants dictionary is passed into the `LayoutNode` initializer and can be referenced by any expression in that node or any of its children.

A good use for constants would be localized strings, or something like colors or fonts used by the current app theme. These are things that never (or rarely) change during the lifecycle of the app, so its acceptable that the view hierarchy must be torn down in order to reset them.

Here is how you would pass some constants inside `LayoutViewController`:

	self.loadLayout(
	    named: "MyLayout.xml",
	    constants: [
	    	"title": NSLocalizedString("homescreen.title", message: ""),
	    	"titleColor": UIColor.primaryThemeColor,
	    	"titleFont": UIFont.systemFont(ofSize: 30),
	    ]
	)

And how you might reference them in the XML:

	<UIView ... >
		<UILabel
			width="100%"
			textColor="titleColor"
			font="{titleFont}"
			text="{title}"
		/>
	</UIView>

(You may have noticed that the `title` and `font` constants are surrounded by `{...}` braces, but the `titleColor` constant isn't. This is explained in the Expressions section below.)

## State

For more dynamic layouts, you may have properties of the view that need to change frequently, perhaps even during an animation, and recreating the entire view hierarchy to change these is neither convenient nor efficient. For that, you can use state. State works exactly the same way as constants, except you can update state after the `LayoutNode` has been initialized:

	self.loadLayout(
	    named: "MyLayout.xml",
	    state: [
	    	"isSelected": false,
	    ],
	    constants: [
	    	"title": ...
	    ]
	)
	
	func setSelected() {
		self.layoutNode.state = ["isSelected": true]
	}
	
Note that you can used both constants and state in the same Layout, but you should avoid duplicating keys between the state and constants. Do not be tempted to try to override a constant value using the state object - this will result in an error.

As with constants, state values can be passed in at the root node of a hierarchy and accessed by any child node. If children in the hierarchy have their own state properties then these will take priority over values set on their parents. It is generally discouraged to inject state at multiple levels in a node hierarchy however, as it becomes difficult to keep track of where values are coming from, and may reduce opportunities for performance optimizations in the future.

Setting the `state` property of a `LayoutNode` after it has been created will trigger an update. The update causes all expressions in that node and its children to be re-evaluated. In future it may be possible to detect if parent nodes are indirectly affected by the state change of their children and update them too, but currently that is not implemented. This is another reason why you are encouraged to set state only on the root node of a given hierarchy.

In the example above, we've used a dictionary to store the state values, but `LayoutNode` supports the use of arbitrary objects for state. A really good idea for layouts with complex state requirements is to use a `struct` to store the state. When you set the state using a `struct` or `class`, Layout uses Swift's introspection features to compare changes and determine if an update is necessary.

Internally the `LayoutNode` effectively just treats the struct as a dictionary of key/value pairs, but you get to take advantage of compile-time type validation when manipulating your state programmatically in the rest of your program:

	struct LayoutState {
		let isSelected: Bool
	}

	self.loadLayout(
	    named: "MyLayout.xml",
	    state: LayoutState(isSelected: false),
	    constants: [
	    	"title": ...
	    ]
	)
	
	func setSelected() {
		self.layoutNode.state = LayoutState(isSelected: false)
	}

## Actions

For any non-trivial view you will need to bind actions from controls in your view hierarchy to your view controller, and communicate changes back to the view.

You can define actions on any `UIControl` subclass using `actionName="methodName"` in your XML, for example:

    <UIButton touchUpInside="wasPressed"/>
    
Layout uses a little-known feature of iOS called the *responder chain* to broadcast that action up the view hierarchy. It will then be intercepted by whichever is the first parent view or view controller that implements a compatible method, in this case:

    func wasPressed() {
        ...
    }
    
The actions's method name follows the Objective-C selector syntax, so if you wish to pass the button itself as a sender, use a trailing colon in the method name:

	<UIButton touchUpInside="wasPressed:"/>
	
Then the corresponding method can be implemented as:

    func wasPressed(_ button: UIButton) {
        ...
    }

A downside of this approach is that no error is generated if your action method is misnamed - it will simply fail to be called when the button is pressed. It's possible that a future version of Layout will detect this situation and treat it as an error.

## Outlets

The corresponding feature to action binding is *outlets*. When creating views inside a Nib or Storyboard, you typically create references to individual views by using properties in your view controller marked with the `@IBOutlet` attribute.

This mechanism works pretty well, so Layout copies it wholesale, but with a few small enhancements. To create an outlet binding for a layout node, just declare a property of the correct type on your `LayoutViewController`, and then reference it using the `outlet` constructor argument for the `LayoutNode`:

    class MyViewController: LayoutViewController {
    
    	var labelNode: LayoutNode!
    
        public override func viewDidLoad() {
        	super.viewDidLoad()
        	
        	self.layoutNode = LayoutNode(
        		view: UIView(),
        		children: [
        			LayoutNode(
        				view: UILabel(),
        				outlet: #keyPath(self.labelNode),
        				expressions: [ ... ]
        			)
        		]
        	)
        }
    }
    
In this example we've bound the `LayoutNode` containing the `UILabel` to the `labelNode` property. A few things to note:

* There's no need to use the `@IBOutlet` attribute for your outlet property, but you can do so if you feel it makes the purpose clearer
* The type of the outlet property can be either `LayoutNode` or a `UIView` subclass that's compatible with the view used in the node. The syntax is the same in either case - the type will be checked at runtime, and an error will be thrown if it doesn't match up.
* In the example we have used Swift's `#keyPath` syntax for the outlet value for better static validation. This is recommended, but not required.
	
It is also possible to specify outlet bindings when using XML templates as follows:

	<UIView>
		<UILabel
			outlet="labelNode"
			text="Hello World"
		/>
	</UIView>

In this case we lose the static validation provided by `#keyPath`, but Layout still performs a runtime check and will throw a graceful error in the event of a typo or type mismatch, rather than crashing.

## Delegates

Another common pattern used commonly in iOS views is the *delegate* pattern. Layout also supports this, but it does so in an implicit way that may be confusing if you aren't expecting it.

When loading a layout XML file, or a programmatically-created `LayoutNode` hierarchy into a `LayoutViewController`, the views will be scanned for delegate properties and these will be automatically bound to the `LayoutViewController` *if* it conforms to the specified protocol.

So for example, if your layout contains a `UIScrollView`, and your view controller conforms to the `UIScrollViewDelegate` protocol, then the view controller will automatically be attached as the delegate for the view controller. It's that simple!

	class MyViewController: LayoutViewController, UITextFieldDelegate {
    
    	var labelNode: LayoutNode!
    
        public override func viewDidLoad() {
        	super.viewDidLoad()
        	
        	self.layoutNode = LayoutNode(
        		view: UIView()
        		children: [
        			LayoutNode(
        				view: UItextField(), // delegate is automatically bound to MyViewController
        				expressions: [ ... ]
        			)
        		]
        	)
        }
        
        func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        	textField.resignFirstResponder()
        	return false
    	}
    }

There are a few caveats to watch out for, however:

* This mechanism currently only works for properties called "delegate" or "dataSource". These are the standard names used by UIKit components, but if you have a custom control that uses a different name for its delegate, it won't work automatically.

* The binding mechanism relies on Objective-C runtime protocol detection, so it won't work for Swift protocols that aren't `@objc`-compliant.

* If you have multiple views in your layout that all use the same delegate protocol, e.g. several `UIScrollView`s or several `UITextField`s then they will *all* be bound to the view controller. If you are only interested in receiving events from some views and not others, you will need to add logic inside the delegate method implementations to determine which view is calling it. That may involve adding additional outlets in order to distinguish between views.

## Composition

For large or complex layouts, you may wish to split your layout into multiple files. This can be done easily when creating a `LayoutNode` programmatically, by assigning subtrees of `LayoutNode`s to a temporary variable, but what about layouts defined in XML?

Fortunately, Layout has a nice solution for this. Any layout node in your XML file can contain an `xml` attribute that references an external XML file. This reference can point to a local file, or even a remote URL:

	<UIView xml="MyView.xml"/>
	
The attributes of the original node will be merged with the external node once it has loaded. Any children of the original node will be replaced by the contents of the loaded node, so you can insert a placeholder view to be displayed while the real content is loading:

	<UIView backgroundColor="#fff" xml="MyView.xml">
		<UILabel text="Loading..."/>
	</UIView>


# Expressions

The most important feature of the `LayoutNode` class is its built-in support for parsing and evaluating expressions. The implementation of this feature is built on top of the [Expression](https://github.com/nicklockwood/Expression) framework, but Layout adds a number of extensions in order to support arbitrary types and layout-specific logic.

Expressions can be simple, hard-coded values such as "10", or more complex expressions such as "width / 2 + someConstant". The available operators and functions to use in an expression depend on the name and type of the property being expressed, but in general all expressions support the standard decimal math and boolean operators and functions that you find in most C-family programming languages.
	
Expressions in a `LayoutNode` can reference constants and state passed in to the node or any of its parents. They can also reference the values of any other expression defined on the node, or of supported property of the view:

    5 + width / 3
    isSelected ? blue : gray
	min(width, height)
	a >= b ? a : b
	pi / 2
	
Additionally, a node can reference properties of its parent node using `parent.someProperty`, or of its immediate sibling nodes using `previous.someProperty` and `next.someProperty`.

## Layout Properties

The set of expressible properties available to a `LayoutNode` depends on the view, but every node supports the following properties at a minimum:

	top
	left
	bottom
	right
	width
	height
	
These are numeric values (measured in screen points) that specify the frame for the view. In addition to the standard operators, all of these properties allow values specified in percentages:

	<UIView right="50%"/>
	
Percentage values are relative to the width or height of the parent `LayoutNode` (or the superview, if the node has no parent). The expression above is equivalent to writing:

	<UIView right="parent.width / 2">
	
Additionally, the `width` and `height` properties can make use of a virtual variable called `auto`. The `auto` variable equates to the content width or height of the node, which is determined by a combination of three things:

* The `intrinsicContentSize` property of the view, if specified
* Any AutoLayout constraints applied to the view by its (non-Layout-managed) subviews
* The enclosing bounds for all the children of the node.

If a node has no children and no intrinsic size, `auto` is equivalent to `100%`.

Though entirely written in Swift, the Layout library makes heavy use of the Objective-C runtime to automatically generate property bindings for any type of view. The available properties therefore depend on the view class that is passed into the `LayoutNode` constructor (or the name of the XML node, if you are using XML layouts).

Only types that are visible to the Objective-C runtime can be detected automatically. Fortunately, since UIKit is an Objective-C framework, most view properties work just fine. For ones that don't, it is possible to manually expose these using an extension on the view (this is covered below under Advanced Topics).

Because it is possible to pass in arbitrary values via constants and state, Layout supports referencing almost any type of value inside an expression, even if there is no way to express it as a literal.

Expressions are strongly-typed however, so passing the wrong type of value to a function or operator, or returning the wrong type from an expression will result in an error. Where possible, these type checks are performed immediately when the node is first created so that the error is surfaced immediately.

The following types of property are given special treatment in order to make it easier to specify them using a expression string:

## Strings

It is often necessary to use literal strings inside an expression, and since expressions themselves are typically wrapped in quotes, it would be annoying to have to used nested quotes every time. For this reason, string expressions are treated as literal strings by default, so in this example...

	<UILabel text="title"/>
	
...the `text` property of the label has been given the literal value "title", and not the value of the constant called "title", as you might expect.

To use an expression inside a string property, escape the value using `{ ... }` braces. So to use the "title" constant instead, you would write this:

	<UILabel text="{title}"/>
	
You can use arbitrary logic inside expression blocks, including maths and boolean comparisons. The value of the expressions need not be a string, as the result will be *stringified*. You can use multiple expression blocks inside a single string expressions, and mix and match expression blocks with literal segments:

	<UILabel text="Hello {name}, you have {n + 1} new messages"/>
	
## Colors

Colors can be specified using CSS-style rgb(a) hex literals. These can be 3, 4, 6 or 8 digits long, and are prefixed with a `#`:

	#fff // white
	#fff7 // 50% transparent white
	#ff0000 // red
	#ff00007f // 50% transparent red

You can also use CSS-style `rgb()` and `rgba()` functions. For consistency with CSS conventions, the red, green and blue values are specified in the range 0-255, and alpha in the range 0-1:

	rgb(255,0,0) // red
	rgba(255,0,0,0.5) // 50% transparent red
	
You can use these literals and functions as part of a more complex expression, for example:

	<UILabel textColor="isSelected ? #00f : #ccc"/>

	<UIView backgroundColor="rgba(255, 255, 255, 1 - transparency)"/>
	
The use of color literals is convenient for development purposes, but you are encouraged to define constants for any commonly uses colors in your app, as these will be easier to refactor later. 
	
## Images

Static images can be specified by name or via a constant or state variable. As with strings, to avoid the need for nested quotes, image expressions are treated as literal string values, and expressions must be escaped inside `{ ... }` braces:

	<UIImageView image="default-avatar"/>
		
	<UIImageView image="{imageConstant}"/>
	
	<UIImageView image="image_{index}.png"/>


## Fonts

Like strings and images, font properties are treated as a literal string and expressions must be escaped with `{ ... }`. Fonts are a little more complicated however, because the literal value is itself a space-delimited value that can encode several distinct pieces of data.

The `UIFont` class encapsulates the font family, size, weight and style, so a font expression can contain any or all of the following space-delimited attributes, in any order:

	bold
	italic
	condensed
	expanded
	monospace
	<font-name>
	<font-size>
	
The foont name is a string and font size is a number. Any attribute that isn't specified will be set to the system default - typically 17pt San Francisco. Here are some examples:

	<UILabel font="bold"/>
	
	<UILabel font="Courier 15"/>
	
	<UILabel font="Helvetica 30 italic"/>

These literal properties can be mixed with inline expressions, so for example to override the weight and size of a `UIFont` constant called "themeFont" you could use:

	<UILabel font="{themeFont} {size} bold"/>

## Attributed Strings

Attributed strings work much the same way as regular string expressions, except that you can use inline attributed string constants to create styled text:
	
	loadLayout(
	    named: "MyLayout.xml",
	    constants: [
	    	"styledText": NSAttributedString(string: "styled text", attributes: ...)
	    ]
	)
	
	<UILabel text="This is some {styledText} embedded in unstyled text" />
	
However, there is a really cool extra feature built in to attributed string expressions - they support inline HTML markup:

	LayoutNode(
	    view: UILabel(),
	    expressions: [
	    	"text": "I <i>can't believe</i> this <b>actually works!</b>"
	    ]
	)

Using this feature inside an XML attribute would be awkward because the tags would have to be escaped using `&gt` and `&lt;`, so Layout lets you use HTML inside a view node, and it will be automatically assigned to the `attributedText` expression:

	<UILabel>This is a pretty <b>bold</b> solution</UILabel>
	
Any lowercase tags are interpreted as HTML markup instead of a `UIView` class. This  relies on the built-in `NSMutableAttributedString` HTML parser, which only supports a very minimal subset of HTML, however the following tags are all supported:
	
	<p>, // paragraph
	<h1>, <h2>, etc // heading
	<b>, <strong> // bold
	<i>, <em> // italic
	<u> // underlined
	<ol>, <li> // ordered list
	<ul>, <li> // unordered list
	<br/> // linebreak
	
And as with regular text attributes, inline HTML can contain embedded expressions, which can themselves contain either attributed or non-attributed string variables or constants:

	<UILabel>Hello <b>{name}</b></UILabel>
	

# Advanced Topics

## Manual Integration

If you would prefer not to subclass `LayoutViewController`, you can mount a `LayoutNode` directly into a view or view controller by using the `mount(in:)` method:
	
	class MyViewController: UIViewController {
    	
    	var layoutNode: LayoutNode!
    	
        public override func viewDidLoad() {
        	super.viewDidLoad()
        	
        	// Create a layout node from and XML file or data object
        	self.layoutNode = LayoutNode.with(xmlData: ...)
        	
        	// Mount it
        	try! self.layoutNode.mount(in: self)
        }
        
        public override func viewWillLayoutSubviews() {
        	super.viewWillLayoutSubviews()
        	
        	// Ensure layout is resized after screen rotation, etc
        	try! self.layoutNode.update()
        }
    }

Note that both the `mount(in:)` and `update()` methods will throw an error. An error will be thrown if there is an error in your layout expression syntax, logic or XML markup. These errors typically only happen if you have made a mistake in your code, so it should be OK to suppress them with `!` during development. If you are loading XML templates from a external source, you may wish to catch and log them instead.

This method of integration does not provide the automatic live reloading feature for local XML files, nor the "red box" debugging interface - both of those are implemented internally by the `LayoutViewController`.

## Custom Components

As mentioned above, Layout uses the Objective-C runtime to automatically detect property names and types for use with expressions. The Objective-C runtime only supports a subset of possible Swift types, and even for Objective-C types, some runtime information is lost. For example, it's impossible to detect the valid set of values and names for UIKit enums.

There are also cases where properties may be exposed in a way that doesn't show up as a property at runtime, or the property setter may not be compatible with KVC (Key-Value-Coding), resulting in a crash when it is accessed using `setValue(forKey:)`.

To solve this, it is possible to manually expose additional properties and custom setters/getters for views using an extension. The Layout framework already uses this feature to expose enum constants for many of the common UIKit enums, but if you are using a 3rd party component, or creating a custom one, you may need to write an extension to properly support configuration via Layout expressions.

To generate a property type and setter for a custom view, create an extension as follows:

	extension MyView {
		
		open override class var expressionTypes: [String: RuntimeType] {
			var types = super.expressionTypes
			types["myProperty"] = RuntimeType(...)
			return types
		}
		
		open override func setValue(_ value: Any, forExpression name: String) throws {
			switch name {
			case "myProperty":
				self.myProperty = values as! ...
			default:
				try super.setValue(value, forExpression: name)
			}
		} 
	}
	
These two overrides add "myProperty" to the list of known expressions for that view, and provide a static setter method for the property.

The `RuntimeType` class shown in the example is a type wrapper used by Layout to work around the limitations of the Swift type system. It can encapsulate information such as the list of possible values for a given enum, which it is not possible to determine automatically at runtime.

`RuntimeType` can be used to wrap any Swift type, for example:

	RuntimeType(MyStructType.self)
	
It can also be used to specify a set of legitimate enum values:

	RuntimeType([
        "left": NSTextAlignment.left.rawValue,
        "right": NSTextAlignment.right.rawValue,
        "center": NSTextAlignment.center.rawValue,
    ])

For an enum, you can choose to use either the enum values themselves or the `rawValue`s if they exist. If the type of the property matches the `rawValue` (as is the case for most Objective-C APIs) then it's typically not necessary to also provide a custom `setValue(forExpression:)` implementation, but you'll have to determine this on a per-case basis.


# Example Projects

There are two example projects includes with the Expression library. These use CocoaPods for integration, however the pod directories are included in the repository, so they should be ready to run.

## SampleApp

The SampleApp project demonstrates a range of Layout features. It is split into four tabs, and the entire project, including the `UITabBarController`, is specified in a single Layout.xml file with a single view controller to manage the layout. The tabsPare as follows:

* Boxes - demonstrates use of state to manage an animated layout
* Pages - demonstrates using a `UIScrollView` to create paged content
* Text - demonstrates Layout's text features, include the use of HTML and attributed string constants
* Northstar - demonstrates how Layout can be used to build a real-world layout using the Northstar components

## UIDesigner

The UIDesigner project is an experimental WYSIWYG tool for constructing layouts. It's written as an iPad app which you can run in the simulator or on a device.

UIDesigner is currently in a very early stage of development. It supports most of the features exposed by the Layout XML format, but lacks import/export, and the ability to specify constants or outlet bindings.