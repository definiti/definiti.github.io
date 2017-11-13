---
title: Definiti
layout: default
---

<div class="lead" markdown="1">

# A descriptive language for your project

> Definiti is a declarative language to define and explicit rules and constraints of your domain.

An important part of your work is to make sure your domain rules and constraints remains valid.
Does your favorite language help you doing so?

What if you could have a language helping you **describing** your **domain**?
What if it was **designed** to do what you want to do: to **verify** your **domain definition**?

It is the challenge of *Definiti*:

* Designed to help you writing the **domain definition**
* Compiled to your favorite language so you focus on what really needs attention

</div>

# A descriptive language

Just describe what you want to do.

```
package my.event

type Event {
  name: String verifying NonEmptyString("Please provide an event name")
  content: Content
  date: Date
  registrationPeriod: Period
  
  verify {
    "The registration period should be before the date of the event"
    (event) => {
      event.registrationPeriod.end.isBeforeOrEqual(event.date)
    }
  }
}

type Content = String verifying NonEmptyString

verification NonEmptyString {
  "Please provide a non empty string"
  (string: String) => {
    string.nonEmpty()
  }
}

type Period {
  start: Date
  end: Date
  
  verify {
    "start and end are reversed"
    (period) => {
      start.isBeforeOrEqual(end)
    }
  }
}
```

# Compiling into other languages

Definiti will compile into your favorite language.

```scala
// Language: Scala

package my
import definiti.native._
import java.time.LocalDateTime
package object event {
  object NonEmptyString {
    def apply(message: String = "Please provide a non empty string"): Verification[String] = Verification(message)((string: String) => StringExtension.nonEmpty(string))
  }
  case class Event(name: String, content: String, date: LocalDateTime, registrationPeriod: my.event.Period)
  object Event {
    val nameVerification: Verification[String] = Verification.traverse(my.event.NonEmptyString("Please provide an event name"))
    val contentVerification: Verification[String] = Verification.traverse(my.event.Content.ContentVerifications, my.event.NonEmptyString())
    val dateVerification: Verification[LocalDateTime] = Verification.traverse()
    val registrationPeriodVerification: Verification[my.event.Period] = Verification.traverse(my.event.Period.allVerifications)
    val EventVerifications: Verification[Event] = Verification.traverse(Verification("The registration period should be before the date of the event")((event: my.event.Event) => DateExtension.isBeforeOrEqual(event.registrationPeriod.end, event.date)))
    val allVerifications: Verification[Event] = Verification.traverse(nameVerification.from((x: Event) => x.name), contentVerification.from((x: Event) => x.content), dateVerification.from((x: Event) => x.date), registrationPeriodVerification.from((x: Event) => x.registrationPeriod)).andThen(EventVerifications)
    def applyCheck(name: String, content: String, date: LocalDateTime, registrationPeriod: my.event.Period): Validation[Event] = allVerifications.verify(Event(name, content, date, registrationPeriod))
  }
  object Content {
    val ContentVerifications: Verification[String] = Verification.traverse(my.event.NonEmptyString())
    def applyCheck(input: String): Validation[String] = ContentVerifications.verify(input)
  }
  case class Period(start: LocalDateTime, end: LocalDateTime)
  object Period {
    val startVerification: Verification[LocalDateTime] = Verification.traverse()
    val endVerification: Verification[LocalDateTime] = Verification.traverse()
    val PeriodVerifications: Verification[Period] = Verification.traverse(Verification("start and end are reversed")((period: my.event.Period) => DateExtension.isBeforeOrEqual(period.start, period.end)))
    val allVerifications: Verification[Period] = Verification.traverse(startVerification.from((x: Period) => x.start), endVerification.from((x: Period) => x.end)).andThen(PeriodVerifications)
    def applyCheck(start: LocalDateTime, end: LocalDateTime): Validation[Period] = allVerifications.verify(Period(start, end))
  }
}

```

# So you can use them as you want

Use it with your favorite framework, library and everything you already use.

```scala
// Language: scala, with scalatest

class MyEventTest extends FlatSpec with Matchers {
  "An Event" should "be valid" in {
    val input = Event(
      name = "Great Opening!",
      content = "I want to describe it but it is just a test",
      date = LocalDateTime.parse("2018-01-01T20:30:00"),
      registrationPeriod = Period(
        start = LocalDateTime.parse("2017-01-01T00:00:00"),
        end = LocalDateTime.parse("2017-12-31T00:00:00")
      )
    )
    val validation = Event.allVerifications.verify(input)
    validation should === (Valid(input))
  }

  "An Event" should "be in because of its registrationPeriod" in {
    val input = Event(
      name = "Great Opening!",
      content = "I want to describe it but it is just a test",
      date = LocalDateTime.parse("2018-01-01T20:30:00"),
      registrationPeriod = Period(
        start = LocalDateTime.parse("2017-12-31T00:00:00"),
        end = LocalDateTime.parse("2017-01-01T00:00:00")
      )
    )
    val validation = Event.allVerifications.verify(input)
    validation should === (Invalid(Seq("start and end are reversed")))
  }
}
```

# Getting start

```bash
# Download sample project
$ git clone -b getting-start https://github.com/definiti/samples.git getting-start

# Go to the project folder
$ cd getting-start

# Run the project, it will build the compiler and build the project
$ nut run
```

See the complete [Getting start](/doc/getting-start) guide for more information.

Afterward, see the [Language reference](/doc/language).

# See us on Github

All the sources are available on Github.

Check the [core compiler](https://github.com/definiti/definiti-core),
[samples](https://github.com/definiti/samples)
and the [Definiti api](https://github.com/definiti/definiti-api).

If you want to improve this project, do not hesitate to fork any repository and write issues.