---
title: The Definiti Project
layout: default
---

Definiti is composed of a language and a project.
The language documentation is available on left menu.
This page describes the overall project.

* TOC
{:toc}

# What is The Definiti Project?

The Definiti Project is a long-term project to give software actors tools to write and maintain software.

Today, the world is full of talks, documentations, languages, IDE, best-practices, integration tools, etc.
All of them give answers of specific issues and should not be blindly accepted or rejected.

How could we advise to go in one direction and not in another?
Experience shows that functional programming was mainly refused at first and came back with great consideration.
And this is only one example in thousands.

In the past ten years, how many times did you need to learn a new language, framework, library, architecture, …?
Each time, did the time lost doing so worth it? The answer is not as simple as yes or no.

Since the beginning of software development, lot of best-practices came out.
A few of them would be **Clean code**, **Domain Driven Development**, **Agile Software Development**, **Software Craftsmanship**…
They bring lot and lot of ideas, answers in generic terms so it can be adapted to most of projects.

Go through the internet, mainstream tools seems not answering domain-level issues but keep answering technical issues.
Most of failed project fail because of project perspective, not technical one.

The Definiti Project tries to correct this gap between project and technique.

# Is the language the answer?

No. The Definiti language is **an** answer.

It tries to show software actors that it is possible to defines the domain with descriptive code that can be compiled.
It provides another vision, another way to do things.
It is not *the* answer because we do not know what the future is made of.
Maybe someone will create something better, more in line with everybody needs.
In this case, it will be a victory for us, not a loss.

It is also a reason why the Definiti language remains simple and extensible across plugins.

# History of the project

This part explains how the project was born.

## A common issue in all projects

Since we develop software, we all did some things in all projects.
The first one is checking the domain rules.

An actor (human or not) give inputs. We *verify* and *process* these inputs and *give outputs*.
These are mainly the steps of all projects today.
Of course, it is more complex and can have more steps, but the great lines remain the same.

Here, we want to implements the *verify* part.
The objective is to defines a structure to verify inputs and to return information to the actor when there is an error.
The process part is ignored at first.

## Check the internet

When we check the internet to find answers, we find lot of them.
We will go through some.

### Example

To understand what everything is about, we will answer the following need.

> As an administrator of the software,
> I want to create a new theater.
> This theater will have:
>
> * A name (required)
> * An address (required, in one-line, no need to be specific)
> * One or more plans
>
> A plan in the theater is composed of:
>
> * A stage
> * A list of seats, all uniquely named
>
> Each one of these elements are place in the plan

First of all, we will define a model of our data:

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

We can have other propositions but we will stay with this.

⚠️ Any code is considered as being the better one.
They show issues with solutions, not how to do the best code in a solution.

### Imperative verification

This verification system is the one most of us has firstly learn.
The objective is to check is one data is invalid and returning a message. 

```typescript
// Language: TypeScript

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

function verifyPlan(plan: Plan): string | undefined {
  const stagePositionError = verifyPosition(plan.stage)
  if (stagePositionError) {
    return stagePositionError
  } else {
    const seatsErrors = plan.seats.map(verifySeats).filter(error => !!error)
    if (seatsErrors.length > 0) {
      return seatsErrors[0]
    } else if (seatsAreUnique(plan.seats)) {
      return undefined
    } else {
      return "Please provide only unique seats"
    }
  }
}

function verifyPosition(position: Position): string | undefined {
  if (position.x < 0) {
    return "The x-axis is invalid"
  } else if (position.y < 0) {
    return "The y-axis is invalid"
  } else {
    return undefined
  }
}

// functions verifySeats, seatsAreUnique, isEmpty ignored
```

In this case, how many time do you need to retrieve information about the domain rule?
What happen in case there is several errors? The actor would need to resend the object several times.

A variant is to use exception at the first error instead of returning the string across functions.

### Smart setters

In object-oriented language, the same rule was repeated:

> All your attributes must be private

And when we push this further, we will have smart setters.

```java
// Language: Java

class Theater {
    private String name;
    private String address;
    private List<Plan> plans;
    
    // constructor and getters
    
    void setName(String name) {
        if (name != null && name.length > 0) {
            this.name = name;
        } else {
            throw new InvalidArgumentException("Please provide a theater name");
        }
    }
    // idem address
    
    void setPlans(List<Plan> plans) {
        if (plans != null && plans.length > 0) {
            this.plans = plans;
        } else {
            throw new InvalidArgumentException("Please provide a theater name");
        }
    }
}

class Plan {
    private Position stage;
    private List<Seat> seats;
    
    // constructor, getters and setStage
    
    void setSeats(List<Seats> seats) {
        if (seats != null && seats.length > 0) {
            if (this.seatsAreUnique(seats)) {
                this.seats = seats;
            } else {
                throw new InvalidArgumentException("Please provide only unique seats");
            }
        } else {
            throw new InvalidArgumentException("Please provide at least one seat");
        }
    }
}

// ignored: Seat and Position
```

This solution ensures you have only valid objects.
But the cost of maintenance, of ensuring that all exceptions are catch and readability is quite disturbing.

And another question needs an answer: if two attributes depend on each other, how to validate both when setting them
and how to authorize a temporary invalid state because of this dependency?

### Annotation

Some frameworks provide a mean to externalize these verifications with the help of annotations.

```java
// Language: Java
class Theater {
    @NotNull
    @Size(min=1)
    private String name;
    
    @NotNull
    @Size(min=1)
    private String address;
 
    @NotNull
    @Size(min=1)
    @Valid
    private List<Plan> plans;
}

class Plan {
    @NotNull
    @Valid
    private Position stage;
    
    @NotNull
    @Valid
    @SeatsAreUnique
    private List<Seat> seats;
}

@java.lang.annotation.Documented
@ConstraintValidator(value = SeatsAreUniqueValidator.class)
@java.lang.annotation.Target(value = {java.lang.annotation.ElementType.FIELD})
@java.lang.annotation.Retention(value = java.lang.annotation.RetentionPolicy.RUNTIME)
public @interface SeatsAreUnique {
  String message() default "Please provide only unique seats";
  String[] groups() default {};
}

public class SeatsAreUniqueValidator implements ConstraintValidator<SeatsAreUnique, List<Seat>> {

  public void initialize(SeatsAreUnique constraintAnnotation) {
  }

  public boolean isValid(List<Seat> object, ConstraintValidatorContext constraintContext) {
    if (object == null)
      return false;
    
    return this.seatsAreUnique(seats);
  }
  
  // ignored: seatsAreUnique
}
```

This code is pretty readable in the bean object.
You have your list of constraints for each field.
However, the readability of `SeatsAreUniqueValidator` is not that simple, take time to understand and to write.
And why should there is two files, one for the annotation, the other for the validator?
And why should we precise `@Valid` each time?

By experience, it is really easy to forgot one thing and having our validations that do not work.
But the logic is interesting.

### Monad

These last years, the term monad come out everywhere.
Not so much developers know what it is and think it is something complex, weird, for alien.
The reality is much more simple and most of them already use them without knowing it.
But it is not our subject.

In this example, I will not use all the monad theory, but will keep it very simple.

```scala
// I repeat here the same model that previously.
case class Theater(name: String, address: String, plans: List[Plan])
case class Plan(stage: Position, seats: List[Seat])
case class Seat(name: String, position: Position)
case class Position(x: Int, y: Int)

// When we valid something, it return Valid or Invalid with the list of errors.
// We suppose there are methods to compose validations but not shown here.
sealed trait Validation
case class Invalid(errors: List[String]) extends Validation
case object Valid extends Validation

object TheaterValidation {
  def verifyTheater(theater: Theater): Validation = {
    // I check that all validations are valid
    Validation.verifyAll(
      StringValidation.validateNonEmpty(theater.name),
      StringValidation.validateNonEmpty(theater.address),
      ListValidation.validateNonEmpty(theater.plans)
    ) andThen {
      // In case previous validations are all valid, we verify all plans are valid
      Validation.verifyList(theater.plans.map(verifyPlans))
    }
  }
  
  def verifyPlan(plan: Plan): Validation = {
    Validations.verifyAll(
      verifyPosition(plan.stage),
      ListValidation.validateNonEmpty(plan.seats)
    ) andThen {
       Validation.verifyList(plan.seats.map(verifySeat))
    } andThen {
      if (seatsAreUnique(plan.seats)) {
        Valid
      } else {
        Invalid(List("Please provide only unique seats"))
      }
    }
  }
  
  // ignored: verifyPosition, verifySeat
}
```

Because of the composition system, the validation is quite simple to write and read for trained eyes.
But the composition pushed too far can also become quite complex to read, even though the compiler guide us quite well.

In this code, I see the domain rules, but all the composition system is pollution for the domain understanding.
It is here only for technical reasons.

And when your model is quite complex, composition become also quite complex

### Other means

There is lot of other mean to validate data.
We cannot show them all but some are quite interesting.
We remain in mainstream and generic solutions.
Be free to search the subject.

## The same issue

In all previous systems, there is always the same issue.
All system try to use a language to apply verifications.

Languages need to compile in a language understandable to a machine.
They are generic enough to answer all needs, verifications, processing, side-effect, other…
So it is complex for them to answer specific needs like verifications (they should do it for everything, not quite simple).

What I want as a software actor is to say what my business rule is, not write exceptions, annotations or monads.

## The birth of The Definiti Project

Because of this pain to not be able to be expressive enough in verifying my business rules, The Definiti Project was born.
The objective is to offer software actors other tools to express their business.

The first objective was to offer a language being all about verifications.
The implicit objective bound to the first one was the language should not replace others but work with them.
The processing part, side-effects will not be managed by this language.
It is a little an objective about Separation Of Concerns.

# The story is just starting

The Definiti Project is just starting.

Though years, we hope to offer more and more tools, helps to help the community to improve in software development.
To know what are following projects, please check the [roadmap](./roadmap).