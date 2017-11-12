---
title: Roadmap
layout: default
---

This page describes the main roadmap of the Definiti Project.
It is not a day-by-day plan, but more of an overview.
Also, objectives are not ordered by importance, chronology or anything.

* TOC
{:toc}

# Project class diagrams

With all packages and types, it is now possible to generate diagrams.
Diagrams are not the answer of good understanding of a project but can help in getting relation quickly.
Moreover, they can show lack of organisation in some cases.

The objective is having a generator creating diagrams in several levels:

* Project level
* Package level
* Aggregate level

# Glossary

Sometime, understanding a domain is complex because of its terms.
Experience shows that a lack of tool to document a project drive to a lack of documentation.

The objective is to give software actor a simple tool to generate a glossary based on the code.

Tools already exist in several languages but are not always known and usable for all needs.

# Javascript compiler

The javascript compiler already exist, but it need to be resynchronized with last version of the language

# Test language

Tests are good. But good tests are complex to write, maintain, understand.
The objective is to search if there is a syntax possible to test domain rules simply.

Because of `context` syntax, we can have a completely different language dedicated to tests if needed.

# Http language

Today, more and more system use http as protocol to communicate across application.
With REST, Falcor, GraphQL, it become complex to keep understanding application and knowing how to navigate an API.
Swagger gives answer for REST api, but could we have something more generic?

Also, if we can describe a http api with verifications, maybe we can generate client api
executing client-side verifications before going to the network and improving latency.

# Json validation

Directly bounded with [http language](#http-language), validating JSON data will become a requirement.

# Form validation

Currently, form validation is often duplicated with server-side verification.
Because of types and verifications, maybe we can improve form validation.