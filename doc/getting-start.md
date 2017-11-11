---
title: Getting start
layout: default
---

This page explains how to get started to initialize the project and run the compiler.

* TOC
{:toc}

# Download

The project does not provide a direct installation.
We use [nut](https://github.com/matthieudelaro/nut) to contain all the project.

Please check the project and follow installation instruction: https://github.com/matthieudelaro/nut

# Initialize workspace

After installing [nut](https://github.com/matthieudelaro/nut), we can prepare the workspace.

The simplest thing to do is to download the [getting-start](https://github.com/definiti/samples/tree/getting-start) sample:

```bash
# Download the project
$ git clone -b getting-start https://github.com/definiti/samples.git getting-start

# Go to the project folder
$ cd getting-start

# Run the project, it will build the compiler and build the project
$ nut run
```

This project is configured to:

* Read your Definiti source files from `src`
* Generate scala source files into `build`

Your workspace is ready!

# Further reading

You can:

* [Read the language documentation](./language)
