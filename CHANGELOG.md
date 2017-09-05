# Change Log

## [0.4.20](https://github.com/schibsted/layout/releases/tag/0.4.20) (2017-09-05)

- Fixed a change in the `LayoutNode.bind()` implementation that cuased a regression in one of our projects

## [0.4.19](https://github.com/schibsted/layout/releases/tag/0.4.19) (2017-09-05)

- Layout now compiles without modification in Xcode 9 using Swift 3.2 or 4.0
- Fixed a bug with disappearing UIAlertController buttons in iOS 9 (introduced in 0.4.14)
- Improved scrolling performance for auto-sized UITableView cells
- Improved UITableView auto height calculation when using multi-line labels
- UITableView section headers and footers no longer vanish after calling `reloadData` 
- Fixed some bugs in parsing of keyPaths in expressions
- Improved expression parsing performance in Swift 3.2 and above

## [0.4.18](https://github.com/schibsted/layout/releases/tag/0.4.18) (2017-09-01)

- Improved support for `UISearchBar`, `UISegmentedControl` and `UIStepper`
- Added customization options for `UINavigationController`'s navigation bar and toolbar
- Added customization options for `UITabBarController`'s tab bar
- Improved API for handling view constructor arguments and made it public
- Improved documentation for view and view controller integration
- Fixed potential array bounds crash when adding views to a view controller
- Added support for class property expressions

## [0.4.17](https://github.com/schibsted/layout/releases/tag/0.4.17) (2017-08-28)

- You can now use `//` comments inside layout expressions
- Empty expressions are now permitted (they are treated the same as omitting the expression)
- Table cell `auto` height is now calculated correctly for cells using multiline `textLabel`
- The `state` property is now deprecated. Use `setState()` instead
- You can now call animated variants of property setters using `setState(_:animated:)`
- Default width is now capped at 100% of the parent width for views such as labels
- Layout now correctly handles custom classes when app name contains spaces or punctuation
- Throwing an error inside `createView()` for custom view subclasses no longer crashes

## [0.4.16](https://github.com/schibsted/layout/releases/tag/0.4.16) (2017-08-25)

- Fix for spurious "property unavailable" errors when using `UITableView`, `UICollectionView` or `UIStackView`
- Added `UIStackView` documentation to README.md

## [0.4.15](https://github.com/schibsted/layout/releases/tag/0.4.15) (2017-08-22)

- Reverted a change to constant evaluation order in 0.4.14 that caused a regression in one of our projects
- Added CONTRIBUTING.md

## [0.4.14](https://github.com/schibsted/layout/releases/tag/0.4.14) (2017-08-19)

- Cells can now access constants and state defined in their containing `UITableView` or `UICollectionView`
- `UITableView` and `UICollectionView` now support `auto` correctly inside width and height expressions
- Fixed a bug where errors in `UITableView` and `UICollectionView` cells were not reported
- Named color constants defined as static properties of `UIColor` can now be used in color expressions
- Added `.layout-ignore` file feature for excluding directories from XML-file search

## [0.4.13](https://github.com/schibsted/layout/releases/tag/0.4.13) (2017-08-17)

- Added support for custom parameters for XML templates
- Fixed a bug where `UITableView` section headers were displayed as gray bar when not specified
- `UIControl` actions and outlets now work correctly inside `UITableView` and `UICollectionView` cells
- Fixed bug that prevented views from other modules from being used inside `UITableViewCell` templates
- Improved error messaging, particularly for view properties that unavailable in Layout
- LayoutTool format now performs additional validation of expressions
- Fixed XML parsing performance regression
- Fixed bug where LayoutTool errors were returning a generic message instead of the actual error
- Improved source file lookup performance by ignoring hidden directories (such as .git)

## [0.4.12](https://github.com/schibsted/layout/releases/tag/0.4.12) (2017-08-14)

- Fixed scroll performance when using a UIScrollView as the root node in a Layout hierarchy
- Added support for UITableViewController and UICollectionViewController

## [0.4.11](https://github.com/schibsted/layout/releases/tag/0.4.11) (2017-08-14)

- Non-ascii characters such as emoji are now encoded and displayed correctly when using inline HTML
- LayoutTool rename command can now rename view or controller classes, as well as expression symbols
- Improved LayoutTool error reporting - error messages now include the file in which the error ocurred
- LayoutTool errors are now displayed in Xcode when running LayoutTool as a build phase script
- LayoutTool errors will now fail the build when running LayoutTool as a build phase script

## [0.4.10](https://github.com/schibsted/layout/releases/tag/0.4.10) (2017-08-11)

- Fixed bug that caused file conflicts to be presented as Red Box error overlay instead of the file selection overlay

## [0.4.9](https://github.com/schibsted/layout/releases/tag/0.4.9) (2017-08-10)

- Fixed assertion failure when repeatedly pressing Cmd-R in the simulator
- Fixed bug when creating subclasses of `UITableView`, `UITableViewCell`, `UITableViewController` or `UICollectionView`
- Added support for manipulating `CGAffineTransform` and `CATransform3D` properties using expressions
- Fixed some cases where `value(forSymbol:)` returned nil instead of throwing an error for invalid properties 
- Fixed a bug where `UICollectionView.collectionViewLayout.itemSize` was ignored by default
- Assigning values to properties of a nil object now fails silently instead of throwing an error

## [0.4.8](https://github.com/schibsted/layout/releases/tag/0.4.8) (2017-08-09)

- Added support for the `UIScrollView.contentInsetAdjustmentBehavior` property introduced in iOS 11
- Property accessor now throws an error for non-existent properties instead of returning nil
- Fixed compiler errors in Xcode 9b5

## [0.4.7](https://github.com/schibsted/layout/releases/tag/0.4.7) (2017-08-07)

- First open source release!
- Added support for UICollectionView
- Improved error messaging
- Fixed a bug where AutoLayout support would not work correctly in some cases

## [0.4.6](https://github.com/schibsted/layout/releases/tag/0.4.6) (2017-08-03)

- Added support for UIStackView
- Empty XML files no longer crash Layout
- Changed styling of file disambiguation dialog to avoid confusion with Red Box error dialog
- Added Cmd-alt-R hard reset keyboard shortcut for clearing user file selection
- Unsupported HTML tags now raise an error
- Auto-sizing calculations are now more tolerant of recursive width/height references
- Fixed a circular reference crash in width/height expressions
- Fixed a bug with Red Box error not updating after a reload
- The first two subviews added to a UITableView now become the header and footer respectively
- Fixed an intermittent crash due to reloading in the middle of expression evaluation
- Added Sandbox example app

## [0.4.5](https://github.com/schibsted/layout/releases/tag/0.4.5) (2017-07-28)

- Added basic templating functionality for inheriting common layout attributes and elements
- Nested xml file references now permit the root node of the referenced file to be a subclass
- Datasource and delegate properties can now be set explicitly, or cleared with `nil`
- Errors thrown when evaluating constant expressions are now correctly propagated
- LayoutTool now has a rename function for renaming expression symbols inside xml files
- Better handling of implicitly-unwrapped Optionals in constants or state
- Improved support for selectors and CGRect properties
- Fixed crash in XML parser when handling a malformed layout structure
- Fixed crash in UIDesigner example

## [0.4.4](https://github.com/schibsted/layout/releases/tag/0.4.4) (2017-07-25)

- Selected source paths are now persisted between application launches
- Made the `LayoutNode.bind(to:)` and `LayoutNode.unbind()` methods public
- More helpful errors for empty expressions

## [0.4.3](https://github.com/schibsted/layout/releases/tag/0.4.3) (2017-07-20)

- Added support for `UITableViewHeaderFooterView` layouts for table footers and headers
- You can now define `UITableViewCell` and `UITableViewHeaderFooterView` templates inside the XML for a table
- Enabled setting the properties of a `UITableViewCell.contentView` and `backgroundView` using expressions
- Errors inside `UITableViewCell` layouts now appear in red box instead of being silently discarded
- Added support for percentage-based font sizes
- Improved performance when using nested XML file references by avoiding redundant view creation
- Optimized view property update performance
- Using CGColor and CGImage constants now works correctly
- Added better support for CoreFoundation and CoreGraphics types
- Further improvements to auto-sizing behavior
- Fixed bug where Red Box scrollView delegate could get bound to a custom controller

## [0.4.2](https://github.com/schibsted/layout/releases/tag/0.4.2) (2017-07-17)

- Improved auto-sizing behavior for views using both `intrinsicContentSize` and AutoLayout constraints
- XML Layouts are now cached in memory after first load, removing the overhead of repeated parsing
- Added performance tests for XML loading, and switched performance tests to release mode

## [0.4.1](https://github.com/schibsted/layout/releases/tag/0.4.1) (2017-07-15)

- Added support for dynamic text resizing based on user font size settings
- Improved font expression parsing. Now handles font names containing spaces, even without quotes
- Fixed an occasional glitch where auto-sized table cells would all be positioned at the top of the table

## [0.4.0](https://github.com/schibsted/layout/releases/tag/0.4.0) (2017-07-12)

- Breaking change: `UIScrollView.contentInset` is now taken into account when using `auto` in width/height
- Using `auto` sizing on nodes whose children depend on parent size no longer creates circular reference errors
- Spurious errors are no longer thrown during mounting of the `LayoutNode` due to premature evaluation
- Setup of `LayoutNode` is now deferred until first update, which reduces unnecessary processing
- Layout now automatically detects Obj-C setter methods and allows them to be used as expression properties
- The `LayoutViewController` class now conforms to the `LayoutLoading` protocol
- Fixed a bug introduced in version 0.3.4 that affected AutoLayout-based view sizing
- Added caching for computed properties, which improves update performance
- Fixed live reloading of strings

## [0.3.6](https://github.com/schibsted/layout/releases/tag/0.3.6) (2017-07-10)

- Conflict resolution now ignores xml or strings files inside build directory
- Made conflict resolution urls more readable by removing common path prefixes
- Fixed bug where error red box would not update after the error was fixed
- Improved support for standard Cocoa struct types
- LayoutTool now formats expressions as well as general XML structure
- Internalized the Expression library dependency, to simplify integration with LayoutTool

## [0.3.5](https://github.com/schibsted/layout/releases/tag/0.3.5) (2017-07-05)

- Added conflict resolution for when multiple strings or XML layout files have the same name
- UIControl event bindings are now more reliable, and added error detection for misspellings
- Fixed bug where XML validation errors were not displayed
- Added LayoutTool project and tests
- Restructured project and removed Northstar example

## [0.3.4](https://github.com/schibsted/layout/releases/tag/0.3.4) (2017-06-29)

- Added support for dynamic table cell height based on AutoLayout
- Improved AutoLayout integration and fixed a potential crash
- Improved performance by avoiding double-evaluation of frame properties on each layout
- Fixed a bug where a circular reference could cause a crash in some cases instead of showing a red box

## [0.3.3](https://github.com/schibsted/layout/releases/tag/0.3.3) (2017-06-27)

- Added `LayoutLoading` protocol, for loading arbitrary views or view controllers from XML
- LayoutVC errors are no longer passed up to parent, which fixes a bug with reloading after an error
- Significantly improved property setting performance - at least 2X faster
- Loading of nested XML files is now synchronous, avoiding a flash as subviews are loaded
- Improved default frame behavior, and added unit tests
- Added support for `UITableView`, `UITableViewCell`, and `UITableViewController`
- Added custom initializer support for views/controllers that require extra arguments
- Added support for `layoutMargins` in expressions

## [0.3.2](https://github.com/schibsted/layout/releases/tag/0.3.2) (2017-06-20)

- Reloading a nested LayoutViewController now reloads just the current controller, not the parent
- Fixed potential AutoLayout crash

## [0.3.1](https://github.com/schibsted/layout/releases/tag/0.3.1) (2017-06-13)

- You can now reference nested state or constants dictionary keys using keypaths in expressions
- Fixed a major performance regression introduced in 0.2.1, which made the UIDesigner unusable
- Added support for setting CGColor and CGImage properties on a CALayer using expressions
- Fixed string formatting when concatenating optional strings
- Improved null-coalescing operator error messaging
- Improved AnyExpression implementation to eliminate arbitrary limits on numeric values

## [0.3.0](https://github.com/schibsted/layout/releases/tag/0.3.0) (2017-06-09)

- Implicitly unwrapping a nil value inside a string expression will now throw instead of printing "nil"
- String and image expressions can now safely return nil values, all other expression types will throw
- Added `nil` literal value, for checking if values are nil inside an expression
- Added `??` null-coalescing operator, for proving fallback values for nil inputs in expressions
- All properties now use AnyExpression internally, meaning they can reference arbitrary types 
- Braces are now permitted around any expression type, making the rules less confusing to remember

## [0.2.6](https://github.com/schibsted/layout/releases/tag/0.2.6) (2017-06-08)

- Fixed bug where custom enums would be misidentified as the wrong type
- Fixed bug with events being swallowed inside nested view controllers of LayoutViewController

## [0.2.5](https://github.com/schibsted/layout/releases/tag/0.2.5) (2017-06-07)

- Fixed bug with locating xml layout files inside a framework bundle
- Fixed Cmd-R shortcut inside LayoutViewController when presented modally

## [0.2.4](https://github.com/schibsted/layout/releases/tag/0.2.4) (2017-06-06)

- Added support for directly using localized strings from XML, and live loading of edited strings
- Added support for missing UIScrollView enum properties
- Added convenient solution for merging constants dictionaries
- Further optimized startup and update performance  

## [0.2.3](https://github.com/schibsted/layout/releases/tag/0.2.3) (2017-05-29)

- Fixed bug where inherited state incorrectly shadowed local constants
- Expressions that mix math functions with non-numeric types now work as expected
- Improved expression parsing and evaluation performance

## [0.2.2](https://github.com/schibsted/layout/releases/tag/0.2.2) (2017-05-19)

- Fixed a critical bug where Bool properties were not working on 32-bit devices
- Improved performance by not recalculating property types in setters
- Added missing UIButton state setters

## [0.2.1](https://github.com/schibsted/layout/releases/tag/0.2.1) (2017-05-19)

- Setting state to the same value no longer triggers an update
- Added support for common Core Graphics geometry types such as `CGPoint`/`CGSize`
- Added support for `UIEdgeInsets`-type properties, and affine/3D transforms
- Passing an invalid expression name now throws an exception instead of crashing
- Using an Optional or implicitly unwrapped Optional for state no longer fails
- Improved handling of Optionals and nil values in constants and state variables
- Fixed some bugs where constants or state variable names shadowed view properties

## [0.2.0](https://github.com/schibsted/layout/releases/tag/0.2.0) (2017-05-18)

- Improved support for enums - it's no longer necessary to explicitly work with raw values (breaking change)
- Shadowing of inherited constants and state now works in a more intuitive way
- Cyclical references in expressions now correctly throw an error instead of crashing
- You can now use font names containing spaces in font expressions by wrapping them in single or double quotes

## [0.1.1](https://github.com/schibsted/layout/releases/tag/0.1.1) (2017-05-16)

- Added support for single- or double-quoted string literals inside expressions
- Fixed reloading of nested xml file references so it correctly finds the source file
- Improved `auto` sizing logic to better handle `intrinsicContentSize` values

## [0.1.0](https://github.com/schibsted/layout/releases/tag/0.1.0) (2017-05-15)

- First release
