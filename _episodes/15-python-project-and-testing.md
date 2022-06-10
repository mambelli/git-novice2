---
title: Setting up a Python project
teaching: 25
exercises: 0
questions:
- "How should I structure a Python project?"
- "How can I test my code to prevent bugs?"
objectives:
- "Make our repository follow a 'standard' Python project format"
- "Add a test and testing framework"
- "Run the tests and a linter on GitLab - using a branch and a Pull Request"
keypoints:
- "While there is often variation, most Python projects follow a similar structure for their code"
- "Doing so is beneficial because it allows components of your code to be reused more easily by yourself and others"
- "Testing can be 'automatic' rather than manual. This catches many issues before they become a problem - this is continuous integration"
- "The concepts here can be used for all programming languages - not just Python - and are pretty much universally used by professional software developers"
---

# Structuring a Project
Vlad and Wolfsman need some calculation to start sending rockets to the planets. They decided to start a Python project.

Most Python projects are structured in a similar way. There are very good reasons for this - if you follow the 'standard', other people who approach your code will recognise parts of it and will know by default how to install your code, run any tests that might exist, and where to look for source code or to change things like the dependencies that are required.
 
Generally, at a minimum, a Python project has the following structure:

~~~
planetsmath/
├── DEVELOPMENT.md
├── .editorconfig
├── .github
│   └── workflows
│       └── linters.yaml
├── .gitignore
├── LICENSES
│   └── Apache-2.0.txt
├── LICENSE.txt
├── .pre-commit-config.yaml
├── .pylintrc
├── README.md
├── requirements.txt
├── .reuse
│   ├── dep5
│   └── templates
│       └── compact.jinja2
├── setup.py
└── src
    └── planetsmath
        ├── functions.py
        ├── __init__.py
        └── test_functions.py
~~~
{: .output}
You can see this sample project [here](https://github.com/mambelli/planetsmath) and clone it with:
~~~
$ git clone git@github.com:mambelli/planetsmath.git
~~~
{: .bash}

We'll take these folders/files one at a time:

## src/
The source tree where all the Python files reside.

## planetsmath/
Inside the project source tree we normally have a folder which matches the name of the Python module. This is done so that from the src directory , the code within the module can be imported with:
~~~
import planetsmath
~~~
{: .python}
You can see this for e.g. in the source repository of the [NumPy library](https://github.com/numpy/numpy).

Within this folder, we can store code files (e.g. functions.py) and further subdirectories. These will then be importable too.

### functions.py

This is just a standard Python file with methods in it. You can have as many of these as you like, but generally people organise them around what the code is doing. So for e.g if you have a few methods that deal with I/O, you might create a file called `io.py` and put all of those methods there. Organising your code across multiple files like this is a very good idea - it makes it easier to find things.

### `__init__.py`
The `__init__.py` file is effectively as set of instructions that get run when you import a Python module. So with a blank `__init__.py`, nothing happens if you run `import planetsmath` in a Python session. If you want to use methods from the functions.py file. What is common is to import certain methods into the top level of the module, for e.g.:

~~~
from .functions import sum_function
~~~
{: .python}

Then, in a Python session, you would be able to do the following:

~~~
>>> import planetsmath
>>> planetsmath.sum_function([1, 2, 3, 4])
10
~~~
{: .python}

## setup.py

A setup.py file is just a list of instructions for Python that tell it how to install your package, and what it's made up of. There are a myriad of options, but a very simple one for this project could be:

~~~
from setuptools import setup

setup(
        name='planets',
        version='0.0.1',
        packages=['planetsmath'],
        install_requires=[
            'numpy',
        ],
)
~~~
{: .python}

Notice that there is a section called 'install_requires'. This isn't required for our package, as we're not using NumPy, but it is very common to see a list of external packages here. On install with pip, Python will check to see if it can import 'numpy' and 'matplotlib'. If it can't, the installation will fail.

## requirements.txt

This is just a text file where you can put any dependencies your package needs to work. If necessary, you can constrain some of your package dependencies to specific versions, for e.g.:

~~~
pytest
numpy>=1.21.4
~~~
{: .output}

To install all of the dependencies, you can run `pip install -r requirements.txt`. This is well known by most people working with Python. Generally you should try to install things via pip like this, and not via Anaconda (unless you have no choice). This is because Anaconda is less portable:
* Is not usable by commercial organisations without a paid for license. This matters if you have external companies as collaborators
* Was introduced mainly for distributing compiled dependencies. This is now well handled by pip with the introduction of `wheels`
* Anaconda is not usable or is heavily discouraged on many HPC clusters.


# Pre-commit checks

It is possible to run some checks before each commit command taking advantage of the hooks mechanism in Git.
A pre-commit will guarantee that committed code always follows the desired standard.
To start using pre-commit you have to add a pre-commit config file and to install pre-commit.

Add a pre-commit config file named .pre-commit-config.yaml with the following content:
~~~
# For more information see
#  https://pre-commit.com/index.html#install
#  https://pre-commit.com/index.html#automatically-enabling-pre-commit-on-repositories
default_language_version:
  # force all unspecified python hooks to run python3
  python: python3
repos:
  - repo: "https://github.com/pre-commit/pre-commit-hooks"
    rev: v4.1.0
    hooks:
      - id: check-ast
      - id: check-docstring-first
      - id: check-toml
      - id: check-merge-conflict
      - id: check-yaml
      - id: end-of-file-fixer
      - id: fix-byte-order-marker
      - id: mixed-line-ending
      - id: trailing-whitespace
        args:
          - "--markdown-linebreak-ext=md"
  - repo: "https://github.com/pre-commit/pygrep-hooks"
    rev: v1.9.0
    hooks:
      - id: python-check-blanket-noqa
      - id: python-check-blanket-type-ignore
      - id: python-use-type-annotations
  - repo: "https://github.com/pycqa/isort"
    rev: 5.10.1
    hooks:
      - id: isort
  - repo: "https://github.com/psf/black"
    rev: 22.3.0
    hooks:
      - id: black
  - repo: "https://github.com/pre-commit/mirrors-prettier"
    rev: v2.6.2
    hooks:
      - id: prettier
        exclude_types:
          - "python"
        additional_dependencies:
          - "prettier"
          - "prettier-plugin-toml@0.3.1"
  - repo: "https://github.com/asottile/pyupgrade"
    rev: v2.31.0
    hooks:
      - id: pyupgrade
        args:
          - "--py36-plus"
  - repo: "https://github.com/fsfe/reuse-tool"
    rev: v0.14.0
    hooks:
      - id: reuse
        additional_dependencies:
          - python-debian==0.1.40
~~~
{: .yaml}

To install it run `pre-commit install` from the repository root.
You may want to setup automatic notifications for pre-commit enabled
repos: https://pre-commit.com/index.html#automatically-enabling-pre-commit-on-repositories

You can also run pre-commit manually to check all the files:
~~~
pre-commit run --all-files
~~~
{: .bash}


# Licensing compliance

Planets Math is released under the Apache 2.0 license and license compliance is
handled with the [REUSE](http://reuse.software/) tool.
REUSE is installed as development dependency or you can install it manually
(`pip install reuse`). All files should have a license notice:

- to check compliance you can use `reuse lint`. This is the command run also by the pre-commit and CI checks
- you can add on top of new files [SPDX license notices](https://spdx.org/licenses/) like
  ```
  # SPDX-FileCopyrightText: 2022 Fermi Research Alliance, LLC
  # SPDX-License-Identifier: Apache-2.0
  ```
- or let REUSE do that for you (`FILEPATH` is your new file):
  ```
  reuse addheader --year 2022 --copyright="Fermi Research Alliance, LLC" \
    --license="Apache-2.0" --template=compact FILEPATH
  ```
- Files that are not supported and have no comments to add the SPDX notice
  can be added to the `.reuse/dep5` file
- New licenses can be added to the project using `reuse download LCENSEID`. Please
  contact project management if this is needed.


# GitHub testing

First, we'll introduce a new file. In the 'planetsmath' subdirectory, in a file called 'test_functions.py' we can write any tests of methods in 'functions.py'. Generally, the library 'pytest' is commonly used for this. Pytest can pick up tests written in the following way:

~~~
from .functions import sum_function

def test_sum_function():
    assert sum_function([1, 2, 3, 4, 5]) == 15.0
    assert sum_function([1, 2.2, 3, 4, 5]) == 15.2
    assert sum_function([-1, 2, 3, 4, 5]) == 13
~~~
{: .python}

Note that both the file name and the method inside are preceded with `test_` - this is compulsory!

With this, when pytest is installed, you can run 'py.test -v' at the command line, and all of your tests will run. With this, you can check your code for correctness.

Create a file called .github/workflows/pytest.yaml with the following contents:
~~~
# SPDX-FileCopyrightText: 2022 Fermi Research Alliance, LLC
# SPDX-License-Identifier: Apache-2.0

---
name: PyTest
on:
  push:
    branches:
      - "**" # matches every branch
  pull_request:
    branches:
      - main

jobs:
  run_linters:
    name: Run unit test against code tree
    runs-on: ubuntu-latest
    steps:
      - name: checkout code tree
        uses: actions/checkout@v2
      - name: Set up Python 3.7
        uses: actions/setup-python@v2
        with:
          python-version: "3.7"
          architecture: "x64"
      - name: Install dependencies
        run: |
          python3 -m pip install --upgrade pip
          if [ -f requirements.txt ]; then python3 -m pip install -r requirements.txt; fi
      - name: Unit test
        env:
          PYTHONPATH: ${{ github.workspace }}/src
        run: |
          python3 -m pytest --import-mode=append src
~~~
{: .yaml}

This is a basic recipe that will allow your tests to run on GitHub. Add both of these files to your staging area and commit them, and then push to GitHub.

