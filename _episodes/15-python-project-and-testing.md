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
Most Python projects are structured in a similar way. There are very good reasons for this - if you follow the 'standard', other people who approach your code will recognise parts of it and will know by default how to install your code, run any tests that might exist, and where to look for source code or to change things like the dependencies that are required.
 
Generally, at a minimum, a Python project has the following structure:

~~~
- planets/
  - planets/
    - functions.py
    - __init__.py # Often this is a 'blank' file.
  - setup.py
  - requirements.txt
  - README.md
~~~
{: .output}

We'll take these folders/files one at a time:

## planets/
Inside the project, we normally have a folder which matches the name of the Python module. This is done so that from the top directory , the code within the module can be imported with:
~~~
import planets
~~~
{: .python}
You can see this for e.g. in the source repository of the [NumPy library](https://github.com/numpy/numpy).

Within this folder, we can store code files (e.g. functions.py) and further subdirectories. These will then be importable too.

### functions.py

This is just a standard Python file with methods in it. You can have as many of these as you like, but generally people organise them around what the code is doing. So for e.g if you have a few methods that deal with I/O, you might create a file called `io.py` and put all of those methods there. Organising your code across multiple files like this is a very good idea - it makes it easier to find things.

### `__init__.py`
The `__init__.py` file is effectively as set of instructions that get run when you import a Python module. So with a blank `__init__.py`, nothing happens if you run `import planets` in a Python session. If you want to use methods from the functions.py file. What is common is to import certain methods into the top level of the module, for e.g.:

~~~
from .functions import sum_function
~~~
{: .python}

Then, in a Python session, you would be able to do the following:

~~~
>>> import planets
>>> planets.sum_function([1, 2, 3, 4])
10
~~~
{: .python}

# setup.py

A setup.py file is just a list of instructions for Python that tell it how to install your package, and what it's made up of. There are a myriad of options, but a very simple one for this project could be:

~~~
from setuptools import setup

setup(
        name='planets',
        version='0.0.1',
        packages=['planets'],
        install_requires=[
            'numpy',
        ],
)
~~~
{: .python}

Notice that there is a section called 'install_requires'. This isn't required for our package, as we're not using NumPy, but it is very common to see a list of external packages here. On install with pip, Python will check to see if it can import 'numpy' and 'matplotlib'. If it can't, installation will fail.

# requirements.txt

This is just a text file where you can put any dependencies your package needs to work. If necessary, you can constrain some of your package dependencies to specific versions, for e.g.:

~~~
pytest
numpy>=1.21.4
~~~
{: .output}

To install all of the dependencies, you can run `pip install -r requirements.txt`. This is well known by most people working with Python. Generally you should try to install things via pip like this, and not via Anaconda (unless you have no choice). This is because Anaconda:
* Is not usable by commercial organisations without a paid for license. This matters if you have external companies as collaborators
* Was introduced mainly for distributing compiled dependencies. This is now well handled by pip with the introduction of `wheels`
* Anaconda is not usable or is heavily discouraged on many HPC clusters, including BlueBEAR.

# GitLab testing

First, we'll introduce a new file. In the 'planets' subdirectory, in a file called 'test_functions.py' we can write any tests of methods in 'functions.py'. Generally, the library 'pytest' is commonly used for this. Pytest can pick up tests written in the following way:

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

Create a file called .gitlab-ci.yml with the following contents:
~~~
image: python:3.8

stages:
  - lint
  - test

before_script:
  - python -V
  - apt-get update -y

flake8:
  stage: lint
  tags:
    - bear-gitlab-runners
  script:
    - pip install flake8
    - flake8 --version
    - flake8 -v

test:
  stage: test
  tags:
    - bear-gitlab-runners
  script:
    - pip install -r requirements.txt
    - py.test -v
~~~
{: .yaml}

This is a basic recipe that will allow your tests to run on GitLab. Add both of these files to your staging area and commit them, and then push to GitLab.
