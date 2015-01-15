# Objective-C Style Guide

This style guide outlines the coding conventions I've used on various projects / teams. I used the [NYT iOS Style Guide](https://github.com/NYTimes/objective-c-style-guide) as a starting point since it had great formatting, covered a breadth of topics, and closely aligned with my opinions on Objective-C style.

## Introduction

Here are some of the documents that informed the style guide. If something isn't mentioned here, it's probably covered in great detail in one of these:

* [The Objective-C Programming Language](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App Programming Guide](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)
* CocoaDevCentral Style Guide [Part 1](http://www.cocoadevcentral.com/articles/000082.php) and [Part 2](http://www.cocoadevcentral.com/articles/000083.php)

## clang-format

Many of the style points in this document can be applied to files automatically using the [clang-format](http://clang.llvm.org/docs/ClangFormat.html) tool and the wonderful [Xcode plugin](https://github.com/travisjeffery/ClangFormat-Xcode). You can find the configuration file and instructions [here](https://github.com/flyingjalapeno/clang-format).


## Table of Contents

* [Dot-Notation Syntax](#dot-notation-syntax)
* [Spacing](#spacing)
* [Conditionals](#conditionals)
* [Ternary Operator](#ternary-operator)
* [Error handling](#error-handling)
* [Methods](#methods)
* [Variables](#variables)
* [Blocks](#blocks)
* [Protocols](#protocols)
* [Categories](#categories)
* [ARC](#arc)
* [Naming](#naming)
* [Public Interfaces](#public-interfaces)
* [Comments](#comments)
* [Init & Dealloc](#init-and-dealloc)
* [Literals](#literals)
* [CGRect Functions](#cgrect-functions)
* [Constants](#constants)
* [Enumerated Types](#enumerated-types)
* [Bitmasks](#bitmasks)
* [Private Properties](#private-properties)
* [Booleans](#booleans)
* [Singletons](#singletons)
* [Image Naming and Asset Catalogs](#image-naming-and-asset-catalogs)
* [File Organization](#file-organization)
* [Project Organization](#project-organization)
* [Other Inspiration](other-inspiration)

## Dot-Notation Syntax

Dot-notation should **always** be used for accessing and mutating properties. Bracket notation is to be used in all other instances.

**For example:**
```objc
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

## Spacing

* Indent using 4 spaces. Never indent with tabs. Be sure to set this preference in Xcode.
* Method braces and other braces (`if`/`else`/`switch`/`while` etc.) always open on the same line as the statement but close on a new line.

**For example:**
```objc
if (user.isHappy) {
//Do something
}
else {
//Do something else
}
```
* There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but often there should probably be new methods.
* `@synthesize` and `@dynamic` should each be declared on new lines in the implementation.

## Conditionals

Conditional bodies should always use braces even when a conditional body could be written without braces (e.g., it is one line only) to prevent [errors](https://github.com/NYTimes/objective-c-style-guide/issues/26#issuecomment-22074256). These errors include adding a second line and expecting it to be part of the if-statement. Another, [even more dangerous defect](http://programmers.stackexchange.com/a/16530) may happen where the line "inside" the if-statement is commented out, and the next line unwittingly becomes part of the if-statement. In addition, this style is more consistent with all other conditionals, and therefore more easily scannable.

**For example:**
```objc
if (!error) {
    return success;
}
```

**Not:**
```objc
if (!error)
    return success;
```

or

```objc
if (!error) return success;
```

### Ternary Operator

The Ternary operator, ? , should only be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an if statement, or refactored into instance variables.

**For example:**
```objc
result = a > b ? x : y;
```

**Not:**
```objc
result = a > b ? x = c > d ? c : d : y;
```

## Error handling

When methods return an error parameter by reference, switch on the returned value, not the error variable.

**For example:**
```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

**Not:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

## Methods

In method signatures, there should be a space after the scope (-/+ symbol). There should be a space between the method segments.

**For Example**:
```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```
## Variables

Variables should be named as descriptively as possible. Single letter variable names should be avoided except in `for()` loops.

Asterisks indicating pointers belong with the variable, e.g., `NSString *text` not `NSString* text` or `NSString * text`, except in the case of constants.

Property definitions should be used in place of naked instance variables unless there is an extremely good reason.  If you need to make your variables private, use an anonymous category as outlined here: [Private Properties](#private-properties)

Direct instance variable access should be avoided except in initializer methods (`init`, `initWithCoder:`, etc…), `dealloc` methods and within custom setters and getters. For more information on using Accessor Methods in Initializer Methods and dealloc, see [here](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6).

 `@synthesize` should not be written out explicitely unless you need to overide the default instance variable name. 

**For example:**

```objc
@interface FJSection: NSObject

@property (nonatomic) NSString *headline;

@end
```

**Not:**

```objc
@interface FJSection : NSObject {
    NSString *headline;
}
```

## Blocks

Blocks should be typedefed so that the type can be easily referenced in other places. Like constants, you should define your blocks as close to the code where they are used, like in the header of the class that uses them.

```objc
typedef void (^FJCompletion)(FJModel *object, NSError *error);
```

Blocks should always be copied when used in properties. 

```objc
@property (nonatomic, copy) FJCompletion completion;
```

Block definitions should omit their return type when possible. Block definitions should omit their arguments if they are void. Block arguments should always be named.

## Protocols

Unless a good reason is given, all protocol methods should be `@required` (the default). Single method protocols are always `@required`. Use `@optional` sparingly to prevent from having to sprinkle  `respondsToSelector` throughout your code.

Protocols can be defined in their own header, or in the header of the class that requires that protocol. Typically delegates should follow the later pattern. 

An example of a Delegate protocol:

```objc
@protocol FJUserViewControllerDelegate <NSObject>;
```

Protocols should always inheret from the NSObject protocol. The first parameter in a protocol method should be the object calling the protocol method:

```objc
@protocol FJUserViewControllerDelegate <NSObject>

- (void)viewController:(FJUserViewController*)controller didUpdateUser:(FJUser*)user;

@end
```

### Naming Delegate Protocols

Delegate protocols should always be named in "notification" style. "…didUpdate…", "…willChange…", "…shouldChange…"


## Categories

Categories should be named for the sort of functionality they provide. Try not to create umbrella categories. They should also be prefixed:

```objc
@interface UIImage (FJImageResizing)
```
Categories are also useful for removing methods that subclasses require out of the public interface:

```objc
@interface FJExampleViewController (FJSubclasses)
```

Alaways name your category files as the "Class+CategoryName":

```objc
FJExampleViewController+FJSubclasses.h
```

## ARC

ARC should be enabled for all new projects. All existing projects should be converted to ARC as soon as possible.

## Naming

Apple naming conventions should be adhered to wherever possible, especially those related to [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508)).

Long, descriptive method and variable names are good.

**For example:**

```objc
UIButton *settingsButton;
```

**Not**

```objc
UIButton *setBut;
```

A 2-4 letter prefix (e.g. `FJ`) should always be used for class names and constants. Constants should be camel-case with all words capitalized and prefixed by the related class name for clarity.

**For example:**

```objc
static const NSTimeInterval FJArticleViewControllerNavigationFadeAnimationDuration = 0.3;
```

**Not:**

```objc
static const NSTimeInterval fadetime = 1.7;
```

Properties and local variables should be camel-case with the leading word being lowercase.

Instance variables should be camel-case with the leading word being lowercase, and should be prefixed with an underscore. This is consistent with instance variables synthesized automatically by LLVM. **If LLVM can synthesize the variable automatically, then let it.**

**For example:**

```objc
@synthesize descriptiveVariableName = _descriptiveYetDifferentVariableName;
```

**Not:**

```objc
id varnm;
```

## Public Interfaces

Your class interfaces should **ONLY** contain methods and properties that need to be exposed publicly. All other code should be in the implementation file. 

All methods and properties should be clearly named (you were doing that all ready, no?), and commented so that documentation can be generated automatically.

## Comments

In general, code should be self-documenting. Comments should be used when neccesary for clarity. Here are a few specific situations:

### Class Interfaces

Always include a high level description in a block comment at the top of the header file. What the class is used for, Why you may use it, etc…

All methods and properties should be clearly named, as mentioned above. However, you should also add Java style comments to be used to generate documentation. The easiest way to do that is to use the [VVDocumenter Xcode plugin](https://github.com/onevcat/VVDocumenter-Xcode).

In addition to the parameters and return values, each method and proiperty should be documented with any default values or behavior that the user must understand to use the API. 

###  Method Implementations

In general, implmeentations should not include comments.  Block comments should generally be avoided, acode should be as self-documenting as possible, with only the need for intermittent, few-line explanations. 

When they are needed, comments should be used to explain **why** a particular piece of code does something. Any comments that are used must be kept up-to-date or deleted.

An example may be a complex algorithm that cannot be broken up easily (something like JSON parsing, or other complex math or logic).

## init and dealloc

`init` methods should be structured like this:

```objc
- (instancetype)init {
    self = [super init]; // or call the designated initalizer
    if (self) {
        // Custom initialization
    }

    return self;
}
```

Only implement dealloc if required. For instance, when unsubscribing from notifications.


## Literals

`NSString`, `NSDictionary`, `NSArray`, and `NSNumber` literals should be used whenever creating immutable instances of those objects. Pay special care that `nil` values not be passed into `NSArray` and `NSDictionary` literals, as this will cause a crash.

**For example:**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**Not:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

## CGRect Functions

When accessing the `x`, `y`, `width`, or `height` of a `CGRect`, always use the [`CGGeometry` functions](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html) instead of direct struct member access. From Apple's `CGGeometry` reference:

> All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**For example:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**Not:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

## Constants

Constants should be kept in the classes/files in which they are used for easy reference. You should not create a "Constants" file and use it as a junk drawer for all constants in the app. For instance, if you define a notification `FJUserDidLogOutNotification`, that should be defined in the header of the class posting the notification.

Constants are preferred over in-line string literals or numbers, as they allow for easy reproduction of commonly used variables and can be quickly changed without the need for find and replace. Constants should be declared as `static` constants and not `#define`s unless explicitly being used as a macro.

As with methods and properties, constants should be kept in the implementation file, if they do not need to be referenced by other classes.

**For example:**

```objc
static NSString * const FJAboutViewControllerCompanyName = @"Flying Jalapeno, LLC";

static const CGFloat FJImageThumbnailHeight = 50.0;
```

**Not:**

```objc
#define CompanyName @"Flying Jalapeno, LLC"

#define thumbnailHeight 2
```

## Enumerated Types

When using `enum`s, it is recommended to use the new fixed underlying type specification because it has stronger type checking and code completion. The SDK now includes a macro to facilitate and encourage use of fixed underlying types — `NS_ENUM()`

**Example:**

```objc
typedef NS_ENUM(NSInteger, FJAdRequestState) {
    FJAdRequestStateInactive,
    FJAdRequestStateLoading
};
```

## Bitmasks

When working with bitmasks, use the `NS_OPTIONS` macro.

**Example:**

```objc
typedef NS_OPTIONS(NSUInteger, FJAdCategory) {
  FJAdCategoryAutos      = 1 << 0,
  FJAdCategoryJobs       = 1 << 1,
  FJAdCategoryRealState  = 1 << 2,
  FJAdCategoryTechnology = 1 << 3
};
```

## Private Properties

Private properties should be declared in class extensions (anonymous categories) in the implementation file of a class. Named categories (such as `FJPrivate` or `private`) should never be used unless extending another class.

**For example:**

```objc
@interface FJAdvertisement ()

@property (nonatomic, strong) GADBannerView *googleAdView;
@property (nonatomic, strong) ADBannerView *iAdView;
@property (nonatomic, strong) UIWebView *adXWebView;

@end
```

## Booleans

Since `nil` resolves to `NO` it is unnecessary to compare it in conditions. Never compare something directly to `YES`, because `YES` is defined to 1 and a `BOOL` can be up to 8 bits.

This allows for more consistency across files and greater visual clarity.

**For example:**

```objc
if (!someObject) {
}
```

**Not:**

```objc
if (someObject == nil) {
}
```

-----

**For a `BOOL`, here are two examples:**

```objc
if (isAwesome)
if (![someObject boolValue])
```

**Not:**

```objc
if ([someObject boolValue] == NO)
if (isAwesome == YES) // Never do this.
```

-----

If the name of a `BOOL` property is expressed as an adjective, the property can omit the “is” prefix but specifies the conventional name for the get accessor, for example:

```objc
@property (assign, getter=isEditable) BOOL editable;
```
Text and example taken from the [Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE).

## Singletons

Singleton objects should use a thread-safe pattern for creating their shared instance.  Unless there is a reason, singleton methods should always be named `sharedInstance` so that classes with singletons are easy to identify.

```objc
+ (instancetype)sharedInstance {
   static id sharedInstance = nil;

   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });

   return sharedInstance;
}
```

This will prevent [possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

## Image Naming and Asset Catalogs

Images should be placed in the asset catalog in all new projects. 

Image names should be named consistently to preserve organization and developer sanity. They should be named as one camel case string with a description of their purpose, followed by the un-prefixed name of the class or property they are customizing (if there is one), followed by a further description of color and/or placement, and finally their state.

**For example:**

* `RefreshBarButtonItem` / `RefreshBarButtonItem@2x` and `RefreshBarButtonItemSelected` / `RefreshBarButtonItemSelected@2x`
* `ArticleNavigationBarWhite` / `ArticleNavigationBarWhite@2x` and `ArticleNavigationBarBlackSelected` / `ArticleNavigationBarBlackSelected@2x`.

Do your beast to communicate these standards to the designer slicing the artwork. Even still, images sliced by the designer might not confom to our standards. If this is the case, you can just change the name in the asset catalog **WITHOUT* changing the file name. That way when your slices are updated, you don't have to rename them again.

## File Organization

### Header Files

Header files should only ever declare the public interface. Private/protected methods should be contained in class continuations inside the implementation file. 

Use the @class directive to forward declare classes referenced in the header instead of importing header files. This prevents compile time issues.

#### Order of Content

Each header file should contain the following in this order:
* Class/Header imports
* `@class` declarations
* External constants (e.g. NSNotification constants)
* External Function Declarations
* Structs / Enums
* Class Delegate Protocols Declarations
* Class Interface
* Class Categories
* Class Delegate Protocols

#### Grouping of Methods and Properties
Methods and properties should be grouped in a sensical way acording to function. Use #pragma marks to group sections for large header files. Common groupings are Initialization, configuration, etc…

### Implementation Files

Within an implementation, all methods should be logically grouped using #pragma marks, including protocol implementations. This allows faster code navigation in Xcode. **Do not use block comments**.

Do not forward declare methods at the top of the file, this no longer neccesary with the latest versions of Xcode.

Do use the anonymous category for any "private" properties.

#### Order of Content
* Class/Header Imports
* Constants (e.g. NSNotification constants)
* Function Implementations
* Anonymous Class Category (for private properties)
* Class Implementation
* Class Categories

#### Grouping and order of class Implementation
* @synthesize / @dynamic (only if required)
* dealloc
* init methods
* class methods
* accessors
* subclass overides
* other groups (including logical functional areas and protocol implementations)

## Project Organization
## Directory Structure
All code files (including header, implementation, nibs, and storyboards) should be kept in a directory called "Source". You shoudl not divide these files into subdirectories.  This does make the project "less browseable" in the finder and command line, but increases flexibility when moving files between groups in Xcode - which is where most developers intereact with the source files. Trying to keep Xcode groups in sync with actual file system directories is not worth the hassle.

## Xcode Project
The organization of files should happen in the Xcode Project file. Your groups should make it easy to find your code and the names should be descriptive to the reader.  Consider your project organization to be part of your documentation.

### Groups
Structure the Project with the following top level groups:

* Source (or "App Name")
	* Application
	* UI
* Tests
* Dependencies
* Frameworks

Within each folder, code should be grouped not only by type, but also by feature for greater clarity.

### Group Specific Organization
#### Application

Place the app delegate, info plist, global.h/.m, prefix-header and any other application level files within the Application Folder (functions, macros).

#### UI

Place all view controllers, story boards, nibs, asset collections and view classes within the UI group. This also inclues any UI specific categories or protocols. Group according to View Controller and/or flow. Group common code (used across several view controllers / flows) in a "Common" group and further subdivide that into specific areas (cells, controls, colleciton view layouts, transitions, etc…). If a view is only used by one view controller, it should **NOT** be placed in the "Common" group.

### Other Top Level Groups

Group all non-UI classes functionally. This includes all models, model controllers, network controllers, categories and protocols. Group according to function. Place any non-application specific external classes within this group. 

## Other Inspiration

The following were also used in the creation of this document:

[Luke Redpath](http://lukeredpath.co.uk/blog/2011/06/28/my-objective-c-style-guide/)
[Github](https://github.com/github/objective-c-conventions)
