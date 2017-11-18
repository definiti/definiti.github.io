---
title: Quick introduction
layout: slides
---

<textarea id="source">

class: title

# Definiti

## Definitively your domain definition

---

class: title

# A simple question

## How do you validate your business rules?

---

# Thinking

## Example – Subject

As an administrator of the software, I want to create a new theater.
This theater will have:

* A name (required)
* An address (required, in one-line, no need to be specific)
* One or more plans

A plan in the theater is composed of:

* A stage
* A list of seats, all uniquely named

All of these elements are place inside the plan

---

# Thinking

## Example – Model

```
Theater(
  name: String,
  address: String,
  plans: List[Plan]
)

Plan(
  stage: Position,
  seats: List[Seat]
)

Seat(
  name: String,
  position: Position
)

Position(
  x: Int,
  y: Int
)
```

---

# Thinking

## Strategy 1 : Imperative verification

```typescript
function verifyTheater(theater: Theater): string | undefined {
  if (isEmpty(theater.name)) {
    return "Please provide a theater name"
  } else if (isEmpty(theater.address)) {
    return "Please provide a theater address"
  } else if (isEmpty(theater.plans)) {
    return "Please provide at least one plan"
  } else {
    const planErrors = theater.plans.map(verifyPlan).filter(error => !!error)
    if (planErrors) {
      return planErrors[0]
    } else {
      return undefined
    }
  }
}
```

---

# Thinking

## Strategy 2 : Smart setters

```java
class Theater {
  private String name;
  private String address;
  private List<Plan> plans;
    
  void setName(String name) {
    if (name != null && name.length > 0) {
      this.name = name;
    } else {
      throw new InvalidArgumentException("Please provide a theater name");
    }
  }
  
  void setPlans(List<Plan> plans) {
    if (plans != null && plans.length > 0) {
      this.plans = plans;
    } else {
      throw new InvalidArgumentException("Please provide at least one plan");
    }
  }
}
```

---

# Thinking

## Strategy 3 : Annotations

```java
class Plan {
    @NotNull
    @Valid // Valid?! Isn't it obvious? 🤔
    private Position stage;
    
    @NotNull
    @Valid
    @SeatsAreUnique
    private List<Seat> seats;
}
```

---

# Thinking

## Strategy 3 : Annotations

```java
@java.lang.annotation.Documented
@ConstraintValidator(value = SeatsAreUniqueValidator.class)
@java.lang.annotation.Target(value = {java.lang.annotation.ElementType.FIELD})
@java.lang.annotation.Retention(value = java.lang.annotation.RetentionPolicy.RUNTIME)
public @interface SeatsAreUnique {
  String message() default "Please provide only unique seats";
  String[] groups() default {};
}
// New file
public class SeatsAreUniqueValidator implements ConstraintValidator<SeatsAreUnique, List<Seat>> {
  public void initialize(SeatsAreUnique constraintAnnotation) {}

  public boolean isValid(List<Seat> object, ConstraintValidatorContext constraintContext) {
    if (object == null) {
      return false;
    } else {
      return this.seatsAreUnique(seats);        
    }
  }
}
```

---

# Thinking

## Strategy 4 : Functional programming, monad… 

```scala
// When we valid something, it returns Valid or Invalid with all errors.
// We suppose there are methods to compose validations but not shown here.
sealed trait Validation
case class Invalid(errors: List[String]) extends Validation
case object Valid extends Validation

def verifyTheater(theater: Theater): Validation = {
  // I check that all validations are valid
  Validation.verifyAll(
    StringValidation.validateNonEmpty(theater.name),
    StringValidation.validateNonEmpty(theater.address),
    ListValidation.validateNonEmpty(theater.plans)
  ) andThen {
    // In case previous validations are all valid,
    // we verify all plans are valid
    Validation.verifyList(theater.plans.map(verifyPlans))
  }
}
```

---

# Thinking

## Strategy * : So many others…

* Railway validation
* Specification pattern
* Formal verification
* …

---

class: title

# What all these strategies share?

--

## Coming from the language *then* adapted for our needs

---

class: title

# Can we do it otherwise?

---

class: title

# Coming: The Definiti Project

---

# Reminder: Subject

As an administrator of the software, I want to create a new theater.
This theater will have:

* A name (required)
* An address (required, in one-line, no need to be specific)
* One or more plans

A plan in the theater is composed of:

* A stage
* A list of seats, all uniquely named

All of these elements are place inside the plan

---

# How do I want to reword this subject?

And of course, with code!

(*Live coding*)

---

# A descriptive and functional language

* Descriptive
* No side-effect by design (even `Date.now`!)
* Inspired from functional programming and object-oriented programing
* Independent from any language and framework

---

# Go further (not implemented)

A **language** dedicated to tests?

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

---

A **language** dedicated to API ?

```
context http {% raw %}{{{{% endraw %}
  action UpdateBlog {
    PUT /blog/:id
    requiring input Blog or invalidate with Error
    requiring authentification or invalidate with Unauthorized
    requiring authorization Admin or invalidate with Forbidden
    requiring existing Blog or invalidate with NotFound
    returning Blog
  }
}}}
```

--

Since we return a `Blog`, cannot we automatically check the returned model is valid?
And to proactively detect any error in the process (latent tests)?

--

Since we describe conditions, cannot we execute validations a first time before calling the server?

---

class: title

# Definiti

## Definitively your domain definition

Thank you for your participation!

Any question?

</textarea>