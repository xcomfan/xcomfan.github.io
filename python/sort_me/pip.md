---
layout: page
title: "Python: pip"
permalink: /sort_me/pip
---

## Scripts Modules Packages, and Libraries

A **module** is a Python file that's intended to be imported into scripts or other modules. It often defines members like classes, functions and variables intended to be used in other files that import it.

A **package** is a collection of related modules that work together to provide certain functionality. These modules are contained within a folder and can be imported just like any other modules. This folder will often contain a special `__init__` file that tells python it's a package, potentially containing more modules nested within sub-folders.

A **library** is an umbrella term that loosely means "a bundle of code". These can have tens or even hundreds of individual modules that can provide a wide range of functionality.

## Getting Started With pip and PyPi

Python's package manager is called pip, and it comes bundled with every recent version of Python. pip allows us to install packages that don't come bundled with the Python standard library.  By default pip searches [PyPi](https://www.pypi.org/) or the **Python Package Index**.

`pip list` will show you all the packages you have installed.

`pip show <package_name>` will give you more details about a specific package.

You can install packages from GitHub using pip using a command similar to: `python -m pip install git+https://github.com/realpython/rptree`

You can use the `-e` option with pip to install a package in editable mode. This allows you to work on the source code while still using the packages as if it were installed normally.

## Using Requirement Files

A **requirements file** is a list of all of a project's dependencies. This includes the dependencies needed by the dependencies. It also contains the specific version of each dependency, specified with a double equals sign `==`.

`pip freeze` will list the current project dependencies to `stdout`.

Thus you can generate a `requirements.txt` file with the command `pip freeze > requirements.txt`

Once you have your requirements files you can install all the needed requirements on a different computer with `pip install -r requirements.txt`

By modifying the `requirements.txt` file to use `>=` instead of `==`, you can tell pip to install the latest stable version of the dependency, with the version specified acting as a minimum.  For example to tell pip to install the latest version of `requests`, but never version `2.23.0` you would use the line `requests>=2.22.0, != 2.23.0`.

To upgrade your installed packages, run the command `pip install --upgrade -r requirements.txt`.

## Production vs Development Dependencies

You may want specific packages in your development environment that you don't want in production. For example you may not need unit testing frameworks in production. For this you can create a `requirements_dev.txt` file similar to the example below where you include all the production packages and then add the ones needed for development.

```text
-r requirements.txt
pytest>=4.2.0
```

It's also helpful to have a requirements file with "locked down" or "known good" versions of dependencies for us in a production environment. This way no call to pip will trigger an upgrade and possibly cause an issue.

To do this just make a copy of your requirements files for example `cp requirements.txt requirements_locked.txt` and change all instances of `>=` in the file to `==`.

## Uninstalling Packages

***Note:*** Pip will not warn you if the package you are trying to uninstall is a dependency for another packages. Its a good idea to use the `git show` command and check the `Required-by:` section of the output to confirm you are not breaking something.

To uninstall just use the command `pip uninstall <package_name>` or `pip uninstall -y <package_name>` if you want to skip the confirmation.

## Alternatives to pip

Check out Conda, Pipevn and Poetry