# Expressions

To describe you code, you need to use expressions. In Definiti, all expressions return a value. The list of accepted expressions will be describe in this page.

## Boolean expression

```text
true | false
```

Returns directly `true` or `false`.

```text
true
false
```

## Number expression

```text
[0-9]+('.'[0-9]+)?
```

Returns directly an number.

```text
1
1.2
12.34
```

## String expression

```text
'"' ('\\"' | .)* '"'
```

Returns directly a raw string.

```text
""
"my string"
"My String with \" escaped"
```

## Reference expression

```text
IDENTIFIER = [a-zA-Z0-9]+
```

Returns the value which the reference targets. References can be:

* Parameter \(function, type, verification, lambda\)
* Function
* Enumeration

```text
def function(param1: String): String {
  param1 // reference to parameter param1
}
```

## Logical expressions

```text
<expression> && <expression>
<expression> || <expression>
```

Composed to boolean expressions to verify either their both valid or one of them is valid.

```text
<expression> && <expression>


value1 && value2 // value1 and value2
value1 || value2 // value1 or value2
```

For your information, `&&` operator have precedence over the `||` operator.

```text
false && true || true = true
false && (true || true) = false
```

## Comparison expressions

```text
<expression> (< | <= | > | >= | == | !=) <expression>
```

Compare two values and return a boolean about this evaluation.

```text
1 < 2  // true
1 <= 2 // true
1 > 2  // false
1 >= 2 // false
1 == 2 // false
1 != 2 // true
```

Types in both left and right expressions for equality expressions \(`==` and `!=`\) require to be the same exact type \(even generics\). In the other case, the compiler will not compile.

Type in both left and right expressions for inequality expressions \(`<`, `<=`, `>` and `>=`\) require to be numbers. There is no syntactic sugar to compare orders of other types \(ex: `Date`\). If you need to compare them, use a conversion function.

If you want to check a value is between two numbers, do it with logical expressions:

```text
1 <= number && number <= 10
```

## Computing expressions

```text
<expression> (+ | - | * | / | %) <expression>
```

Compute a new value from two other values.

```text
1 + 2 // 3
1 - 2 // -1
1 * 2 // 2
1 / 2 // 0.5
1 % 2 // 1
```

Operations `*`, `/` and `%` have precedence over `+` and `-`. Otherwise precedence is from left to right.

```text
1 + 2 * 3 // 7
(1 + 2) * 3 // 9
```

Unlike in some languages, it is not possible to concatenate strings with `+`.

## Not expression

```text
!<expression>
```

Reverse the value of the inner expression.

```text
!true = false
!false = true
```

## Function call

```text
<function name> ([ <generics as identifiers> ])? '(' <arguments as expressions> ')'
```

Call a function with arguments.

```text
// function definition
def duplicate(string: String): String => {
  string.append(string)
}

// Usage
duplicate("") // Give ""
duplicate(duplicate("Test")) // Give "TestTestTestTest"
duplicate(" Value ".trim()) // Give "ValueValue"
```

If the function has generics, you must declare them.

```text
// function declaration
def contains[A](list: List[A], element: A): Boolean => {
  list.contains(element)
}

// Usage
contains[String](stringList, "")
contains[Int](intList, 1)
```

## Attribute call

```text
<expression>.<attribute name>
```

Call an attribute from the returned value of the left expression.

```text
"abc".length // 3
duplicate("abc").length // 6
" abc ".trim().length // 3
```

It can be attributes of:

* Native types
* Declared types
* Enumerations

## Method call

```text
<expression>.<method name>(<arguments as expressions>)
```

Execute a method from the returned value of the left expression.

```text
"abc".repeat(1)
"abc".repeat(someComputation(x, y, z))
"abc".substring("abc".indexOf("a") + 1, "abc".length)
```

It can be methods of native types only.

## Conditions

```text
if (<expression>) { <expression> } (else { <expression> })?
```

Execute some code depending of an expression.

```text
if (iAmMajor) {
  getBeer()
} else {
  getJuice()
}
```

The return type of the condition is the return type of both `if` part and `else` part. In the case the type differs, the return type of the expression will be `Unit`.

Even though the `else` part is optionally, there is few \(if none\) cases where it has a value to avoid it in Definiti.

## Ok/Ko expressions

```text
ok
ko(<arguments expressions>)
```

These expressions exist for the use inside verifications. Their return type is not interesting and is not designed to be used outside verifications or to be combined in other expressions.

They describe a validation or an invalidation with errors.

```text
verification PositiveNumber {
  message("positive.number", Number)
  (value: Number) => {
    if (value >= 0) {
      ok
    } else {
      ko(value)
    }
  }
}
```

## Parenthesis

```text
(<expression>)
```

An expression with parenthesis has precedence over other expressions. It is useful to override the default precedence.

```text
(a || b) && c
(a + b) * c
```

## Lambda

```text
(<parameters>) => { <expression> }
```

Lambdas, also known as anonymous functions, arrow functions and so on, describe a function without naming it.

It can only be used as parameters of functions or methods.

```text
listOfString.map[Int]((value: String) => { value.length })
```

Currently, lambda are very verbose. A future work will be done to avoid unnecessary syntax.

## Order of precedence

If you ask yourself what is the order of precedence of all expressions, it is simply from the last one to the first one \(higher to lower\).

It does not really change for most languages so you should not be lost.

Here ends the list of expressions in Definiti.

