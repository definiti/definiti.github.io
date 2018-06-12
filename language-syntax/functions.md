# Functions

## Function syntax

Functions are a way to structure your code and sometime reuse it. They already exist in nearly all languages.

Their syntax is the following:

```text
def isNonEmpty(string: String): Boolean => {
  string.nonEmpty()
}
```

The parameter type and return type are both required. There is no type inference in function return types.

If you need to use generic types, you can do like the following:

```text
def contains[A](list: List[A], element: A): Boolean => {
  list.contains(element)
}
```

## Return value of a function

In Definiti, all expressions return a value. The return type of the function is the return type of the last expression of the function. There is no `return` keyword you can use.

See [Expressions](expressions.md) for more information.

## Restrictions

Functions can only exist as a first-class citizen. You cannot declare a function inside a type, verification or another function.

