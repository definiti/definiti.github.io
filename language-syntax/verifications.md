# Verifications

## Problematic

In most projects, if not all, we need to define some input validations. Also, sometimes, we want to express some invariants in our models.

There is a lot of means to define these validations. Here some cases in different languages that could be found in projects:

```scala
def nonEmptyString(value: String): Validated[String] = {
  if (value.nonEmpty) {
    Valid(value)
  } else {
    Invalid("The string should not be empty")
  }
}
```

```typescript
// If you do not know TypeScript, consider it as JavaScript with types only.
function isNonEmptyString(value: string): boolean {
  return value !== undefined && value !== null && value !== ''
}
```

```php
function validateNonEmptyString(String $value) {
  if ($value == null || empty($value)) {
    throw new InvalidArgumentException("The string should not be empty");
  }
}
```

```java
class NonEmptyStringVerification extends Verification<String> {
  public String getErrorMessage() {
    return "The string should not be empty";
  }

  public boolean validate(String value) {
    return value != null && !value.isEmpty();
  }
}
```

Each previous variant do the same thing: validating that a string is non empty \(with language specific features sometimes\). We usually have the same thing as input \(the string value to validate\) but several output types \(Validation type, boolean, exceptions, …\) and they are structured in different ways \(functions or classes\).

How could we see that all these codes are validations? Here some tips:

* The function or class name contains terms like `verification` or `validate`
* The return type is `boolean` or `Validated`
* There is exceptions returned
* The code structure is a simple condition or a simple `if/else` statement with a return value for each branch

🤔 Seems like there is a code pattern. Could we introduce a language feature?

## Verification syntax

### Literal message

**Definiti** defines a syntax for verification functions:

```text
verification NonEmptyString {
  "The string should not be empty"
  (value: String) => {
    string.nonEmpty()
  }
}
```

That's all. We use the same pattern that other languages but give it a specific syntax. When compiling into other languages, it will be the role of the compiler to use the specific technical structure in your language.

Here, we use a hard-coded message but you can use a code \(eg: `error.non.empty.string`\) for your translation system too.

### Typed message

Literal messages are good for generic messages but how can you add information in your messages? **Definiti** provides a syntax for that:

```text
verification PositiveNumber {
  // Declare a message with the code "positive.number" and a parameter of type Number
  // For instance, the message could be "The number '{0}' is negative"
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

Now, your message can have more information. You can give has many parameters as you want. The `ok` instruction does not accept any parameter \(the value is valid, that's all\) but `ko` instruction requires as many parameters as the message \(except the message code\) and in the declared types.

You can see that the first parameter is a message code and not the message itself. To enforce the usage of a internationalization system, **Definiti** does no string interpolation for verifications.

Message parameter types can be any type, native or project types. The responsibility of converting them into strings is not managed by **Definiti**. Also, it ensures you to give the right type and right number of types.

### Code parameters

Some verifications can also accept parameters to be more generic. For instance, to declare a verification checking that a number is between a specific range, you can do:

```text
verification IsBetween(min: Number, max: Number) {
  // Message sample: The value should be between {min} and {max}.
  message("is.between", Number, Number)
  (value: Number) => {
    if (min <= value && value <= max) {
      ok
    } else {
      ko(min, max)
    }
  }
}
```

The parameters `min` and `max` are defined for the verification and can be used inside the function. You can use them in the condition as well as the `ko` instruction. It is just parameters like any other.

### Generics

If you need to declare a generic in your verification, you can use following syntax:

```text
verification NonEmptyList {
  "The list should not be empty"
  [A](value: List[A]) => {
    value.nonEmpty()
  }
}
```

## Usage

You can use the verification as follow:

```text
// Some verifications declared anywhere
verification IsNonEmptyString {
  "Please give a string"
  (value: String) => {…}
}
verification MaxLengthOf200 {
  "You cannot give more than 200 characters"
  (value: String) => {…}
}
verification IsBetween(min: Number, max: Number) {
  message("is.between", Number, Number)
  (value: Number) => {…}
}
// Project types are also accepted!
verification IsPersonComplete {
  "The person should be complete"
  (value: Person) => {…}
}

// Defined type
// For a Person to be valid, it also need to respect verification IsPersonComplete
type Person verifying IsPersonComplete {
  // When empty, gives the message "Please give a string"
  // When length > 200, gives the message "You cannot give more than 200 characters"
  firstName: String verifying IsNonEmptyString verifying MaxLengthOf200
  
  // When erroneous, gives the message "Please give a last name" because explicited here
  lastName: String verifying IsNonEmptyString("Please give a last name")
  
  // When erroneous, gives the message message("is.between", 18, 150)
  age: Number verifying IsBetween(18, 150)
  
  // When erroneous, gives the message "Hmm… Are you sure?" (with no parameter)
  numberOfSibling: Number verifying IsBetween(1, 20, "Hmm… Are you sure?")
}

// Alias type
type NonEmptyString = String verifying IsNonEmptyString
```

Please refer to other chapters for more information of each syntax.

