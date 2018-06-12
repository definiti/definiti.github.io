# Types

## Problematic

Most \(if not all\) software projects use data. They need to structure it as well as validate it.

### Information structure

All software projects structure its information. It is necessary to describe, manage and distribute it.

There is a lot of means to write this structure. Here some cases in different languages that could be found in projects:

```scala
case class Article(
  title: String,
  content: String,
  tags: Seq[String]
)
```

```typescript
interface Article {
  title: string
  content: string
  tags: Array<string>
}
```

```php
class Article {
  private $title;
  private $content;
  private $tags;
  
  public function __construct(string $title, string $content, array $tags) {
    $this->title = $title;
    $this->content = $content;
    $this->tags = $tags
  }
  
  public function getTitle(): string {
    return $this->title;
  }
  
  public function setTitle(string $title) {
    $this->title = $title;
  }
  
  // other getters and setters
}
```

```java
public class Article {
  private final String title;
  private final String content;
  private final List<String> tags;

  public Article(String title, String content, List<String> tags) {
    this.title = title;
    this.content = content;
    this.tags = tags;
  }
  
  public String getTitle() {
    return title;
  }
  
  // other getters
}
```

Of course, each solution can be used in other languages and is not always representative of its developer community.

All of these solutions do the same thing: describe some information as a list of attributes. Some solutions are more verbose because of the language restrictions or conventions.

How could we see that these elements are data structures?

* There is a list of attributes
* All attributes are accessible from outside \(directly or indirectly\)
* For each attribute, the same pattern appears: attribute definition, declaration inside constructor, getters and setters \(when a pattern is inexistent, it is inexistent for all attribute\)

So we have a common pattern. We could use for instance the version with the least characters to write.

### Information validation

We looked at how to structure the information. For most of the structures, we can pass to the language syntax. However, there is another notion to look at.

Look at the following code:

```scala
case class Period(
  start: Date,
  end: Date
)
```

What is the issue about this code? Should we consider this type valid as soon as we have two dates?

To ensure the data validation, we could do in several ways:

```scala
class Period private (val start: Date, val end: Date)

object Period {
  def apply(start: Date, end: Date): Either[String, Period] = {
    if (start.isBefore(end)) {
      Right(new Period(start, end))
    } else {
      Left("start > end 😱")
    }
  }
}
```

```typescript
interface Period {
  start: Date
  end: Date
}
function isPeriodValid(period: Period): boolean {
  return period.start < period.end
}
```

```php
class Period {
  private $start;
  private $end;
  
  public function __constsruct(Date $start, Date $end) {
    if ($start->isBefore($end)) {
      $this->start = $start;
      $this->end = $end;
    } else {
      throw new Exception("start > end 😱")
    }
  }
  
  public function setStart(Date $start) {
    if ($start->isBefore($this->end)) {
      $this->start = $start;
    } else {
      throw new Exception("start > end 😱")
    }
  }
  
  // other setters and getters
}
```

```java
class Period {
  private Date start;
  private Date end;
  
  public Period(Date start, Date end) {
    if (start.isBefore(end)) {
      this.start = start;
      this.end = end;
    } else {
      throw new RuntimeException("start > end 😱")
    }
  }
  
  // getters
}
```

One more time, you can use some solutions in other languages.

It answers our question of how to validate data consistency. However, did you see that we check that `start < end` and not that `start <= end`? What happens if this restriction was added afterward and some data in your database are now invalid \(where `start == end`\) but you need to accept the value for any reason? Your code and your data are not compatible anymore and will fail each time you query these values.

To avoid this case, you will surely define another function to deserialize data from your database and remove the validation. It is not an issue **as long as you accept it**. However, it creates some boilerplate that could be avoid.

## Type syntax

### Type declaration

You can declare a type by writing the following:

```text
type Article {
  title: String
  content: String
  tags: List[String]
}
```

You can also give an alias to an existing type:

```text
type ArticleId = String
```

For more information about attribute declaration, please refer to part [Attributes](types.md#attributes).

### Verification reference

If you want to specify a verification that your type must validate, you can do it with a verification reference:

```text
type Period verifying IsValidPeriod {
  start: Date
  end: Date
}

// Could also be in another file
verification IsValidPeriod {
  "start > end 😱"
  (period: Period) => {
    period.start.isBefore(period.end)
  }
}
```

You can also do it with alias types:

```text
type NonEmptyString = String verifying IsNonEmptyString

verification IsNonEmptyString {
  "Please give a non empty string"
  (value: String) => {
    value.nonEmpty()
  }
}
```

Please see page [Verifications](verifications.md#usage) for more information about using them.

### Inline verification

If you want to define the verification inside the type, you can also do it:

```text
type Period {
  start: Date
  end: Date
  
  verify {
    "start > end 😱"
    (period) => {
      period.start.isBefore(period.end)
    }
  }
}
```

Or with alias types:

```text
type NonEmptyString = String {
  verify {
    "Please give a non empty string"
    (string) => {
      string.nonEmpty()
    }
  }
}
```

Note that you do not need \(and cannot\) to give the type of the verification parameter. It is induced by the surrounding type.

### Type parameters

If you want to declare parameters to your types in order to reuse their structure and logic, you can do it this way:

```text
type User(minAge: Number) {
  name: String
  age: NumberHigherThan(minAge)
  roles: ListOfMaxSize[String](5) // Maximum 5 roles
}

type NumberHigherThan(min: Number) = Number {
  verify {
    message("number.higher.than", Number, Number)
    (number) => {
      if (number > min) {
        ok
      } else {
        ko(number, min)
      }
    }
  }
}
```

In concrete usage, top-level structures has few value to have type parameters. They mostly come in handy for alias types to increase the level of abstraction of common elements \(`NumberHigherThan`, `ListOfSize`…\).

### Type verification parameters

You can also specify that you type verification need an external value. In this case, you will declare a named verification with parameters.

```text
type Order {
  id: String
  items: List[OrderItem]
  
  verify withConfiguration(orderIdFormat: String) => {
    "The id should respect the configured format"
    (order) => {
      order.id.matches(orderIdFormat)
    }
  }
}

type OrderItem {
  product: String
  quantity: Number
  
  verify withConfiguration(allowedQuantity: Number) => {
    "The quantity is too important"
    (orderItem) => {
      orderItem.quantity <= allowedQuantity
    }
  }
}
```

When you check the full aggregate of Order by using `withConfiguration`, all verifications with this name will be included into the verification. So in above code, both `withConfiguration` of `Order` and `OrderItem` will be call.

The top-level function in targeted language will require all parameters identified by their name. An error will occur if you use a name with different type in the same aggregate.

### Type parameters or type verification parameters?

You could do what you want in both cases. However, these features were designed for two different cases:

* Type parameters were designed for static invariants inside code, top-level aggregates should not have type parameters
* Type verification parameters were designed for external parameters \(configuration, user settings, options taken from an API…\)

When you use one or the other, please think about where the value comes from and why you define this parameter.

## Attributes

We talked about types but not their attributes specifically.

### Attribute declaration

Their default usage is:

```text
type Article {
  title: String
  content: String
  tags: List[String]
}
```

### Attribute verifications

You can add verification references to attributes:

```text
type Article {
  title: String verifying IsNonEmptyString // Will use default error message
  content: String verifying IsNonEmptyString("Please give a content")
  tags: List[String] verifying IsListOfMinSize(3) verifying HasNoDuplicatedElements
}
```

If you want to add a constraint to each tag, you need to define an alias type:

```text
type Article {
  title: String
  content: String
  tags: List[Tag]
}

type Tag = String verifying IsValidTag
```

Also, if alias type has parameters, you can specify them:

```text
type Article {
  title: String
  content: String
  tags: ListOfMinSize[Tag](3)
}
```

### Attribute usage

In the rest of the code, to access to an attribute, just name it:

```text
type Period verifying IsValidPeriod {
  start: Date
  end: Date
}

// Could also be in another file
verification IsValidPeriod {
  "start > end 😱"
  (period: Period) => {
    period.start.isBefore(period.end)
  }
}
```

All attributes are public and immutable.

## Enumerations

Like most language, Definiti has an enumeration system.

You can describe them like:

```text
enum Days {
  Monday
  Tuesday
  Wednesday
  Thursday
  Friday
  Saturday
  Sunday
}
```

Line returns are not required. You can also write the enumeration like:

```text
enum Days {
  Monday Tuesday Wednesday Thursday Friday
  Saturday Sunday
}
```

You can compare enumeration values between them but you cannot use them another way.

