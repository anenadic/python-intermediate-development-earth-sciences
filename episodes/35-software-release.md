---
title: "Packaging Code for Release and Distribution"
teaching: 0
exercises: 30
questions:
- "How do we prepare our code for sharing as a Python package?"
- "How do we release our project for other people to install and reuse?"
objectives:
- "Describe the steps necessary for sharing Python code as installable packages."
- "Use Poetry to prepare an installable package."
- "Explain the differences between runtime and development dependencies."
keypoints:
- "Poetry allows us to produce an installable package and upload it to a package repository."
- "Making our software installable with Pip makes it easier for others to start using it."
- "For complete control over building a package, we can use a `setup.py` file."
---

## Why Package our Software?

We've now got our software ready to release - the last step is to package it up so that it can be distributed.

For very small pieces of software, for example a single source file, it may be appropriate to distribute to non-technical end-users as source code, but in most cases we want to bundle our application or library into a package.
A package is typically a single file which contains within it our software and some metadata which allows it to be installed and used more simply - e.g. a list of dependencies.
By distributing our code as a package, we reduce the complexity of fetching, installing and integrating it for the end-users.

In this session we'll introduce one widely used method for building an installable package from our code.
There are range of methods in common use, so it's likely you'll also encounter projects which take different approaches.

## Packaging our Software with Poetry

### Installing Poetry

Because we've recommended GitBash if you're using Windows, we're going to install Poetry using a different method to the officially recommended one.
If you're on MacOS or Linux, are comfortable with installing software at the command line and want to use Poetry to manage multiple projects, you may instead prefer to follow the official [Poetry installation instructions](https://python-poetry.org/docs/#installation).

We can install Poetry much like any other Python package, using `pip`:

~~~
$ source venv/bin/activate
$ pip3 install poetry
~~~
{: .language-bash}

To test, we can ask where Poetry is installed:

~~~
$ which poetry
~~~
{: .language-bash}

~~~
/home/alex/python-intermediate-inflammation/venv/bin/poetry
~~~
{: .output}

If you don't get similar output, make sure you've got the correct virtual environment activated.

Poetry can also handle virtual environments for us, so in order to behave similarly to how we used them previously, let's change the Poetry config to put them in the same directory as our project:

~~~ bash
$ poetry config virtualenvs.in-project true
~~~
{: .language-bash}

### Setting up our Poetry Config

Poetry uses a **pyproject.toml** file to describe the build system and requirements of the package.
This file format was introduced to solve problems with bootstrapping packages (the processing we do to prepare to process something) using the older convention with **setup.py** files and to support a wider range of build tools.
It is described in [PEP 518 (Specifying Minimum Build System Requirements for Python Projects)](https://www.python.org/dev/peps/pep-0518/).

Make sure you are in the root directory of your software project and have activated your virtual environment, then we're ready to begin.

To create a `pyproject.toml` file for our code, we can use `poetry init`.
This will guide us through the most important settings - for each prompt, we either enter our data or accept the default.

When we get to the questions about defining our dependencies, we'll answer no, so we can do this separately later.

~~~
$ poetry init
~~~
{: .language-bash}

~~~
This command will guide you through creating your pyproject.toml config.

Package name [example]:  inflammation
Version [0.1.0]: 0.1.0
Description []:  Analyse patient inflammation data
Author [None, n to skip]: James Graham <J.Graham@software.ac.uk>
License []:  MIT
Compatible Python versions [^3.8]: ^3.8

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Generated file

[tool.poetry]
name = "inflammation"
version = "0.1.0"
description = "Analyse patient inflammation data"
authors = ["James Graham <J.Graham@software.ac.uk>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"


Do you confirm generation? (yes/no) [yes] yes
~~~
{: .output}

### Project Dependencies

Previously, we looked at using a `requirements.txt` file to define the dependencies of our software.
Here, Poetry takes inspiration from package managers in other languages, particularly NPM (Node Package Manager), often used for JavaScript development.

Tools like Poetry and NPM understand that there are two different types of dependency: runtime dependencies and development dependencies.
Runtime dependencies are those dependencies that need to be installed for our code to run, like NumPy.
Development dependencies are dependencies which are an essential part of your development process for a project, but are not required to run it.
Common examples of developments dependencies are linters and test frameworks, like `pylint` or `pytest`.

When we add a dependency using Poetry, Poetry will add it to the list of dependencies in the `pyproject.toml` file, add a reference to it in a new `poetry.lock` file, and automatically install the package into our virtual environment.
If we don't yet have a virtual environment activated, Poetry will create it for us - using the name `.venv`, so it appears hidden unless we do `ls -a`.
Because we've already activated a virtual environment, Poetry will use ours instead.
The `pyproject.toml` file has two separate lists, allowing us to distinguish between runtime and development dependencies.

~~~
$ poetry add matplotlib numpy
$ poetry add --dev pylint
$ poetry install
~~~
{: .language-bash}

These two sets of dependencies will be used in different circumstances.
When we build our installable package and upload it to a package repository, Poetry will only include references to our runtime dependencies.
This is because someone installing our software through a tool like `pip` is only using it, but probably doesn't intend 
to contribute to the development of our software and does not require development dependencies.

In contrast, if someone downloads our code from GitHub, together with our `pyproject.toml`, and installs the project that way, they will get both our runtime and development dependencies.
If someone is downloading our source code, that suggests that they intend to contribute to the development, so they'll need all of our development tools.

Have a look at the `pyproject.toml` file again to see what's changed.

### Packaging Our Code

Next, we need to make sure that our code is organised in the recommended Python code package structure.
This is the package (yes, we use the same word to mean two different things...) structure that we encountered in the refactoring section - a directory containing an `__init__.py` and our Python source code files.

We've provided an example of a semi-realistic Python application in the code directory for today.
By convention installable package (the type we install with `pip`) names use hyphens, whereas code package (a directory of Python files) names use underscores.
While we could choose to use underscores in an installable package name, we cannot use hyphens in a code package name, as Python will interpret them as a minus sign when we try to import them.

Once we've got our `pyproject.toml` configuration done and our code in the right structure, we can go ahead and build a distributable version of our software:

~~~
$ poetry build
~~~
{: .language-bash}

This should produce two files for us in the `dist` directory.
The one we care most about is the `.whl` or **wheel** file.
This is the file that `pip` uses to distribute and install Python packages, so this is the file we'd need to share with other people who want to install our software.

Now if we gave this wheel file to someone else, they could install it using `pip` - you don't need to run this command yourself, you've already installed it using `poetry install` above.

~~~
$ pip3 install dist/inflammation*.whl
~~~
{: .language-bash}

The star in the line above is a **wildcard**, that means Bash should use any filenames that match that pattern, with any number of characters in place for the star.
We could also rely on Bash's autocomplete functionality and type `dist/inflammation`, then hit the <kbd>Tab</kbd> key if we've only got one version built.

After we've been working on our code for a while and want to publish an update, we just need to update the version number in the `pyproject.toml` file (using [SemVer](https://semver.org/) perhaps), then use Poetry to build and publish the new version.
If we don't increment the version number, people might end up using this version, even though they thought they were using the previous one.
Any re-publishing of the package, no matter how small the changes, needs to come with a new version number.

~~~
$ poetry build
~~~
{: .language-bash}

In addition to the commands we've already seen, Poetry contains a few more that can be useful for our development process.
For the full list see the [Poetry CLI documentation](https://python-poetry.org/docs/cli/).

The final step is to publish our package to a package repository.
A package repository could be either public or private - while you may at times be working on public projects, it's likely the majority of your work will be published internally using a private repository such as JFrog Artifactory.
Every repository may be configured slightly differently, so we'll leave that to you to investigate.

## What if We Need More Control?

Sometimes we need more control over the process of building our installable package than Poetry allows.
In these cases, we have to use the method that existed before Poetry - a `setup.py` file.
Because this is a Python file, we can use the full power of Python to describe how to setup our project.

One of the common cases where this is particularly useful is if our project has components in different languages.
For example, to speed up some of the core parts we might write some of our functions in C, then call these from our Python code.
Using a `setup.py` gives us the flexibility to handle building these components in different ways and bring them together at the end.

> ## Our Own `setup.py` (Optional)
>
> The [Python Packaging User Guide](https://packaging.python.org/) provides documentation on [how to package a project](https://packaging.python.org/en/latest/tutorials/packaging-projects/) using `setup.py`.
> Using this documentation, can you produce a `setup.py` file for our Poetry project?
>
> Which configuration style do you prefer for projects like this one?
{: .challenge}

{% include links.md %}
