# Native types

## Objective

This chapter will explain the list of native types.

## Problematic

**Definiti** is a language that compile to others. One of its objective is to be compliant with most of languages. To do so, we checked several of them and keep what is common and possible to do in each one.

As lot of tools targeting several technologies, we have to use the lower common denominator. For non existing types, we will explain the reason.

## Existing types

### Boolean

Like in any languages supporting it, a boolean can only have two values : `true` and `false`.

```text
// Examples
true
false
```

### Number

This type represents any kind of numbers, integers and decimals. In generated languages, it should work as a `BigDecimal`.

We did not split this type into several ones to be compliant with JavaScript \(only type `number`\). However, there is lot of drawbacks and we think to changing it in further releases.

```text
// Examples
1
1.2
12.34
-1
-1.2
-12.34
```

### String

A `String` is a representation of a textual information.

All strings in **Definiti** should be considered as encoded in UTF-8. Conversions should be done in targeted languages.

```text
// Example
""
"A string"
"¯\_(ツ)_/¯"
```

### Date

A date is a data containing a temporal information.

All dates in **Definiti** are based on the Gregorian calendar and are in UTC.

{% hint style="warning" %}
The date API could not answer the need of an international application. Please avoid them in these cases. Help defining its API is gladly accepted.
{% endhint %}

No examples, because only accepted from input.

### Option

An `Option` is an object containing either no element or one element. Its purpose it to explicitly give the information that a value could have no value.

In **Definiti**, if a value is not an option, it means that the value _must_ exist. The value `null` does not exist and it is not possible to check the nullity of a value.

An `Option[A]` accept only a value of type  `A` for its internal object.

An `Option` cannot be created in **Definiti**, it must be given as an input. Following code shows a possible syntax if it was possible.

```text
// Example
// Same as null in java, php, javascript…
// Same as None in scala, Nothing in haskell
Option[String]()

// Same as "value" in java, php, javascript…
// Same as Some("value") in scala, Just "value" in haskell
Option[String]("value")
```

### List

A `List` is an object containing from zero to n \(finite\) elements. It is similar to an array, a vector or a list in most languages. There is no derivation of the list. A `List` is always immutable. If an operation is executed on it, it will return a new `List` with updated values.

A `List[A]` accept only a values of type  `A` for its internal values.

A `List` cannot be created in **Definiti**, it must be given as an input. Following code shows a possible syntax if it was possible.

```text
// Example
List[String]()
List[Number](1)
List[String]("some", "words")
```

## Other types not implemented

* Char
* Map
* Tuple
* Stream
* Future / Promise

