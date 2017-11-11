---
title: Language reference
layout: default
---

This page explains the Definiti language and syntax.

* TOC
{:toc}

# Universal rules

These rules are true for every type in the application.

## Every type is a value object

All types are a simple value object.
It means there is no information about its identity (no memory pointer, no primary key, nothing).
So by design, two values are equal if their values are equal.

## Immutable

All types are immutable by design.
Each transformation gives you another instance.

## No side-effect

It is impossible to create a side-effect with this language.
It means that you cannot:

* Read a value of an external context (eg: database connexion, API connexion, `Date.now()`, …)
* Change a value of an external context

Side effect codes will be done in destination languages.

# Types

First of all, like most of languages, your application is composed of types.

## Basic types

These types are provided by the language just for you.
They are mostly classic and similar to other languages.

### Boolean

This type represents a boolean as other languages.
It can have only two values: `true` and `false`.

This type can be used to check condition (`if (boolean)`), reverse condition (`!boolean`)
and compose conditions (`||`, `&&`).

* [Boolean API](https://github.com/definiti/definiti-api/blob/master/src/Boolean.definition)

### Number

This type represents any numeric types.
Currently, there is no difference between integers and decimals numbers.

All numbers are described in decimal literals only (no hexadecimal, no binary or other format).
This type can have mathematical computing operation (`+`, `-`, `*`, `/`, `%`)
and ordering comparison (`<`, `>`, `<=`, `>=`).

* [Number API](https://github.com/definiti/definiti-api/blob/master/src/Number.definition)

### String

This type represents a textual information as in most languages. Its representation is a literal enclosed by `"`.

* [String API](https://github.com/definiti/definiti-api/blob/master/src/String.definition)

### Date

This type represents a moment in time based on the Gregorian calendar.

It is composed of the date and the time. This last one is based on the UTC timezone only.

* [Date API](https://github.com/definiti/definiti-api/blob/master/src/Date.definition)

### List

This type represents an ordered list of elements.
All elements of a list must be of the same type, controlled by a generic type.
It is the equivalent of `Array` in JavaScript, `List` in Java, `Seq` in Scala.

A list provide a lot of methods to read and transform the `List` and its elements.

* [List API](https://github.com/definiti/definiti-api/blob/master/src/List.definition)

### Option

This type represents an optional value.
It means that a value can be defined or not.

It is used in replacement of `null` in Java and `null`/`undefined` in JavaScript.
With this type, you will explicit that a value can not be defined.

All operations on this value will be wrapped by a method so you are guaranteed to always have it defined when manipulating it.

In Definiti, `null` and similar does not exist by design to force use be explicit and using this type.

* [Option API](https://github.com/definiti/definiti-api/blob/master/src/Option.definition)

### Unit

This type represents a result of a computing having any type.
This type should never be used in Definiti.
However, it exists for the compiler to check expressions, types and validate the logic of the application.

If you encounter an error because of a `Unit`, it means an expression does not work as expected.

## Your own types

Basics types are great but they will not describe your domain model.
Here come your own types.

### Defined types

#### Nominal type

A type is composed of several attributes with a type.

For instance:

```
type Blog {
  title: String
  content: String
  tags: List[String]
  published: Boolean
}
```

It defines a type with four attributes: `title`, `content`, `tags` and `published`.
Each attribute have a type: `String`, `String`, `List[String]`, `Boolean`.

For the third type, there is a generic: `[String]`, see section [Generics](#generics) for more information.

So a type just describe a structure as in most language.
But, we could improve the model definition.

#### Verifications on attributes

In the previous example, the type `Blog` can have a `String` as `title`.
But what is the domain rule for this string?

You can add **attribute verifications** (see section [Verifications](#verifications) for more information):

```
type Blog {
  title: String verifying NonEmpty
  //…
}

verification NonEmpty {
  "The string must not be empty"
  (string: String) => {
    string.nonEmpty()
  }
}
```

Here, you explicit that the `title` needs to be `NonEmpty`.

So you directly know if a `Blog` is valid:

```
Blog(title = "") // invalid: The string mist not be empty
Blog(title = "a") // valid
Blog(title = "some text") // valid
```

So you do not need to check another part of your code to check what is the rule for this attribute.
Because it is descriptive, generated code will give you helpers to check your model and returns error messages.

If you want to give a specific message for the validation, you also can do it:

```
type Blog {
  title: String verifying NonEmpty("Please provide a title")
  //…
}
```

Giving you:

```
Blog(title = "") // invalid: Please provide a title
Blog(title = "a") // valid
Blog(title = "some text") // valid
```

#### Verification on type

You can add verifications on attributes, but what about validation of the own type?

It is possible to add **type verifications** inside the type:

```
type Period {
  start: Date
  end: Date
  
  verify {
    "End must be before start"
    // no need to specify type, it is always the enclosing type
    (period) => {
      period.start.isBeforeOrEqual(period.end)
    }
  }
}
```

Here, we check the whole `Period` is valid.

And you can add as many verifications as you want:

```
type FullDayPeriod {
  start: Date
  end: Date
  
  verify {
    "End must be before start"
    (period) => {
      period.start.isBeforeOrEqual(period.end)
    }
  }
  
  verify {
    "Start must start at midgnight"
    (period) => {
      // we simplify it with just hours
      period.start.hour == 0
    }
  }
  
  verify {
    "End must end at midgnight"
    (period) => {
      // we simplify it with just hours
      period.end.hour == 0
    }
  }
}
```

In this case, all three verifications must be valid.

#### External verification on types

If you want to structure you code, you could want to split verifications and type definition.

```
type FullDayPeriod verifying StartsAtMidnight verifying EndsAtMidnight {
  start: Date
  end: Date

  verify {
    "End must be before start"
    // no need to specify type, it is always the enclosing type
    (period) => {
      period.start.isBeforeOrEqual(period.end)
    }
  }
}

verification StartsAtMidnight {
  "Start must start at midgnight"
  // The type must be defined in this case
  (period: Period) => {
    period.start.hour == 0
  }
}

verification EndsAtMidnight {
  "End must end at midgnight"
  // The type must be defined in this case
  (period: Period) => {
    period.end.hour == 0
  }
}
```

But it is kind of boilerplate with `Period` type.
Hopefully, we can refactor it with **alias types**.

### Alias types

**Alias types** are types used to avoid duplicating code.
It take a reference type and add verification on it.

```
type FullDayPeriod = Period verifying StartsAtMidnight verifying EndsAtMidnight
```

Here, the `FullDayPeriod` will be the same type as `Period` and check same verifications.
But to be valid, it should also verify `StartsAtMidnight` and `EndsAtMidnight`.

You can also have an alias for native types:

```
type NonEmptyString = String verifying IsNonEmpty
```

And if you just want to give a meaningful name to a type, you can drop verifications:

```
type BlogId = String
```

Alias types are quite useful to split and clean your code.

### Note on generated codes

Generated codes should provide value object structures with only the type of the attribute.
The generated code will not inform about the list of verifications an attribute or type must valid.
Verifications will be moved to helpers that you can call.

This separation is because of two things:

* Languages have not the verification notion
* If the generated code has a too strong type structure, the code could be unusable

The objective is not to replace the language logic but to work with it.

# Verifications

`Verification` is an important part of your domain definition.

## Objective

> What rules my model should respect?
>
> What verifications should I do?

To answer these questions, other languages gives you techniques based on language features:

* Imperative conditions
* Exceptions is the rule
* Annotations
* Monad
* …

They can all work and can be enough for some applications.
But when your model become complex or you need to duplicate rules in several languages (eg: front-end/back-end),
it become complex to keep everything synchronized and cleaned.

Definiti takes the issue on the other side and provides a unique simple verification system inside the language.

## Defines a verification

A verification is defined as follow:

```
verification IsNonEmpty {
  "The string must not be empty"
  (string: String) => {
    string.nonEmpty()
  }
}
```

We have a `verification` named `IsNonEmpty` which will be callable everywhere in the application.
The default message will be `The string must not be empty`.
The validation accept only `String` (which will be called `string`).
To be valid, the `string` must respect the expression `string.nonEmpty()`.

The syntax is explicit. You can easily understand it and understand what it is about.
If you have explicit `verification` names, your code will be more understandable.

## Usage

To use verifications, you just need to bind it to attributes and types:

```
// Defined type
type MyType verifying MyVerification {
  // …
}

// Alias type
type MyOtherType = MyType verifying MyVerification

// Attribute
type MyType {
  myAttribute: String verifying MyVerification
}
```

But you could also want to add a more useful message than the default one (defined into the `verification`).
You can do it by defining it in parenthesis:

```
// Defined type
type MyType verifying MyVerification("specific message") {
  // …
}

// Alias type
type MyOtherType = MyType verifying MyVerification("specific message")

// Attribute
type MyType {
  myAttribute: String verifying MyVerification("specific message")
}
```

## Note on generated codes

Models will be complex. You will have nested types and nested types.
Generated codes should always verify the whole aggregate and not only the top-level type.

So you should not care about it and just describe your domain rules as good as possible.

# Named functions

You can also defined named functions to reuse code in your application:

```
def periodIsIncluded(period: Period, reference: Period): Boolean => {
  period.start.isAfterOrEqual(reference.start) && period.end.isBeforeOrEqual(reference.end)
}
```

Named functions are top-level elements.
You cannot define a named function inside a type nor another named function.

All named functions must define the return type, there is no type inference here.

A named function can be recursive.
The tail-recursive optimisation will be done either by the generator or delegated to the destination language.
Definiti gives no information about it.

# Expressions

In [verifications](#verifications) and [named functions](#named-functions), there is a body.
This body describes technically how the verification or named function should work.

## Everything is an expression

In Definiti, everything is an expression. There is no `return` keyword and everything returns a value.
When you call an attribute, a method or a condition, the expression will always return something.

## Literal values

These following expressions describe a literal value.
The return type is defined for each case.

```
// Boolean
true
false

// String
"This is my text"

// Number
1
12
12.34
```

## Method calls

When you want to call a method from a value, you need to use the following syntax:

```
string.nonEmpty()
date.plusYears(1)
string.replace("search", "replacement")
```

The value can come from another expression and be chained:

```
string.trim().nonEmpty()
toMidnight(date).plusYears(1)
blog.title.replace("search", "replacement")
```

Parenthesis are required in all cases.

## Attribute calls

When you want to call an attribute from a value, you need to use the following syntax:

```
string.length
date.year
```

The value can come from another expression and be chained:

```
string.trim().length
toMidnight(date).year
blog.title.length
```

## Function calls

When you want to call a function, you need to use the following syntax:

```
myFunction()
myFunction("p1")
myFynction("search", "replacement")
```

## References

When a value exists in scope, it is possible to use it:

```
def isTooLong(string: String, max: Number): Boolean {
  string.length > max
}
```

## Logical expressions

All these expressions returns a boolean value:

```
// Or operator - left and right expressions must be a Boolean
string.isEmpty() || string.startsWith("x")

// And operator - left and right expressions must be a Boolean
string.nonEmpty() && string.endsWith("x")

// Equal operator
"abc" == "abc"

// Not equal operator
"abc" != "abc"

// Ordering operators - left and right expressions must be a Number
x < y
x > y
x <= y
x >= y
```

## Calculator expressions

All these expressions returns a number value:

```
x + y
x - y
x * y
x / y
x % y
```

Because everything is a `Number`, there is no distinction between integers and decimals.
So the `/` on two integers can give a decimal.

## Conditions

You also can use conditions:

```
if (x > y) {
  x
} else {
  y
}
```

If the two inner expressions in `if` and `else` does not provide the same type, the condition returns `Unit`.
It usually means there is an issue with the condition.

For information, the `else` part is optional, but because of the language design, we do not know if it has a use case.
Dropping it usually means there is an issue with the condition.

## Lambda expressions

This expression is special. It can only by used as parameter (function or method).
There is several reasons for it, the first one being language compatibility.

Lambda expressions are like a little function:

```
myList.forAll((element: String) => {
  element.nonEmpty()
})

// Same as

myList.forAll((element: String) => element.nonEmpty())
```

For now, you need to specify the types for each parameter of the lambda.

## Contexts

This feature enables the language to be extensible.
It provides a simple mean for any plugin to add features in a specific box.

For instance, we want to define a new syntax for tests:

```
context test {% raw %}{{{{% endraw %}
  test verification NonEmptyString {
    > When the string is empty, returns an error
    check "" return error
    
    > When the string contains one character, return valid
    check "x" return valid
  }
}}}
```

Or a new syntax for a http API:

```
context http {% raw %}{{{{% endraw %}
  action PostBlog {
    POST /blog
    body: Blog
    authorization: Admin
    returns: Blog
  }
}}}
```

These syntax are not official (and surely does not exist) but they show the possibility provided by the language.

The Definiti project is not to give you another language but to give you tools to implements your domain explicitly.

# Generics

Generics exist in nearly all mainstream languages. This language provides a simplified system of generics.

For instance, `Option` and `List` requires a generic type:

```
myOption: Option[String]
myList: List[Number]
```

With these generic types, you can exploit the type system:

```
myList.forAll((x: Number) => x > 0) // valid
myList.forAll((x: String) => x.nonEmpty) // invalid, myList is a List[Number] 
```

You can define generics in types.

```
type WrapperList[A] {
  inner: List[A]
}
```

You can also use them in verifications.

```
verification IsNonEmpty {
  "The list should not be empty"
  [A](list: List[A]) => {
    list.nonEmpty
  }
}
```

Or in functions.

```
def isNonEmpty[A](list: List[A]): Boolean => {
  list.nonEmpty
}

def first[A](list: List[A]): Option[A] => {
  list.head
}
```

## Advanced generics

There is no advanced generic as covariance, Higher-order types or other things.
The reason is mainly language compatibility.
More research is needed before implementing it.

The Definiti project should remain simple and if there is no need for advanced generics
or if it can be defined otherwise, other solutions could be developed instead.

# Comments

There are two kinds of comments: line comments and block comments.

```
// This is a line comment

/*
  This is a block comment
*/
```

In Definiti, comments are automatically attached to the following element when possible.
It is valid for both line comments and block comments.

```
// Attached to MyType
type MyType {
  // Attached to myAttribute
  myAttribute: String
}

// Attached to MyAlias
type MyAlias = MyType

// Attached to MyVerification
verification MyVerification {
  "The string must not be empty"
  (string: String) => {
    string.nonEmpty()
  }
}

// Attached to myNamedFunction
def myNamedFunction() => {
  true
}
```

For the usual developer, this attachment means nothing, it will not be executed.
But for plugin developers, it means that they do not need to parse the code again, generated AST already contains it.
This is useful for documentation plugins.

Most languages give this two kinds of comments and provide a third kind for documentation (eg:`/** … */` in java).
In Definiti, we do not care about the syntax, all comments are eventual documentation.
Plugins will then use these comments to generate documentation or other things.

# Packages and imports

To split and organize your code, we need to split the code in several files and also organize it for language destination.

## Package definition

Each definiti file can have a single package definition and must be the first non empty line.

```
package my.project

type MyType {
  // …
}
```

This code means that `MyType` is in package `my.project`.
The full name is `my.projet.MyType`.

All top-level elements will be included into the file package.

In generated code, it will depends on the language, but the structure should reflect the package structure.
For instance, in Java, the previous type `MyType` should be into package `my.project`.
In JavaScript, it should be in file `my/project.js` or at least in directory `my/project`.

## Imports

Now you split your code into packages, you need to import top-level elements when you want to use them.

```
package blog

import common.verifications.NonEmptyString
import common.verifications.IsNonEmptyList

type Blog {
  title: NonEmptyString
  tags: List[String] verifying IsNonEmptyList("Please provide at lease one tag")
}
```

In this case, `NonEmptyString` will be recognized as `common.verifications.NonEmptyString`
and `IsNonEmptyList` as `common.verifications.IsNonEmptyList`.

Currently, there is no syntax to merge the two import lines or to rename the element.

Every top-level element can be imported (**verification**, **type** and **named function**).

All **native types** are always automatically imported.

## Directory organisation

The language gives no restriction in the directory structure.
You can have everything in the root of your project or having your directory structure representing the package structure.

For now, we do not give recommendation for a specific organisation.
However, there is three main possible structures.

### Flat

All your Definiti source files are in the root of your directory.

It is relevant for simple projects.

### Directory = package

The directory structure represents the package structure.
It is the conventional structure for Java and Scala projects, for instance.

It is relevant for packages with several files.

### Package with flat leaf directory

The directory structure defines the package structure.
However, in contrary to previous structure, the leaf directory contains several packages.

For instance:

```
/
├── blog
    ├── verifications.def # Package blog.verifications
    └── types.def         # Package blog.types
└── forum
    ├── verifications.def # Package forum.verifications
    └── types.def         # Package forum.types
```

It is relevant when the project is big enough but each package is composed of only one file.

### Other structures

Because we do not force structure, maybe you can find another one that suit your project better.

# Samples

This is the end of the documentation.
If you want to check how to use the language and structure it,
you can check the [samples project](https://github.com/definiti/samples).