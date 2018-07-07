# Definiti

## Describe your domain

Does your language enable you to describe your domain? Does it only describe you domain or do technical stuffs are in your way?

The objective of **Definiti** is to provide a declarative language and tools to help you describe your domain and throw out technical code at the edge of your application.

## How does it work?

Define all your domain logic in **Definiti** language, then generate sources code in your technical languages such as Scala or Typescript.

It is also possible to generate other things than sources files, as glossary, plantUML diagrams and more.

The following diagram represents a simple workflow with **Definiti**:

![](.gitbook/assets/workflow.png)

{% hint style="info" %}
**Definiti** is not intended to work as a standalone language. Please check which languages are supported before using it.
{% endhint %}

## Getting started

{% hint style="warning" %}
We use docker to download and execute the **Definiti** compiler. Please refers to the documentation to install it on your environment.
{% endhint %}

To start using **Definiti**, execute the following installation command:

```bash
bash -c "$(curl https://raw.githubusercontent.com/definiti/definiti/master/script/install.sh -sSf)"
```

You will be ask for the version, the languages and the plugins to use. For instance, we will take following configuration:

* version: _default_
* languages: **scala**
* plugins: _none_

Once done, write your first **Definiti** file in `src/main/definiti/blog.def` \(we will come back to the syntax later\):

{% code-tabs %}
{% code-tabs-item title="src/main/definiti/blog.def" %}
```text
package my.blog

type BlogArticle {
  title: String
  description: String
  date: Date
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Then compile it:

```bash
./definiti
```

You will see a new directory: `target/scalamodel` with the most interesting file `my/blog/blog.scala`:

{% code-tabs %}
{% code-tabs-item title="src/main/scala-definiti/my/blog/blog.scala" %}
```scala
package my

import definiti.native._
import java.time.LocalDateTime

package object blog {
  case class BlogArticle(title: String, description: String, date: LocalDateTime)
  object BlogArticle {
    val verification: Verification[BlogArticle] = Verification.none[BlogArticle]
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can now use it in your own project:

{% code-tabs %}
{% code-tabs-item title="src/main/scala/my/app/App.scala" %}
```scala
package my.app

import java.time.LocalDateTime
import my.blog.BlogArticle


object App {
  def main(args: Array[String]): Unit = {
    val firstArticle = BlogArticle(
      title = "My first article",
      description = "My description",
      date = LocalDateTime.now()
    )
    println(firstArticle)
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## What's next?

Please check the left panel to go through the documentation.

