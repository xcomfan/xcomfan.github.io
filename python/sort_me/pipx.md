---
layout: page
title: "Deploy Your Python Script on the Web With Flask"
permalink: /sort_me/flask_web_deploy
---

## Overview

Reason for using `pipx` over `PyPI` is `pipx` allows you to install and execute runnable scripts without impacting your Python interpreter.

## Get Started with `pipx`

Unlike `pip`, `pipx` doesn't install packages into your system-wide Python interpreter or even an activated virtual environment. Instead, it automatically creates and manages virtual environments for you to isolate the dependencies of every package that you install. `pipx` also adds symbolic links to your `PATH` vairable for every command-line script exposed by the installed packages.

You can classify code distributed through `PyPI` into three catagories

1. **Importable:** It's either pure Python source code or Python bindings of compiled shared objects that you want to import in your Python projects. Typically its libraries such as `requests` providing re-usable pieces of code to help you solve a common problem. They can also be frameworks such as `FastAPI` that you build your applications around.

2. **Runnable:** These are usually command-line utility tools such as `black` or `flake8` that assist you during the development phase. They can also be full-fledged applications like `JupyterLab`

3. **Hybrid:** These combine both worlds by providing importable code and runnable scripts at the same time. `Flask` and `Django` are good examples, as they offer utility scripts while also being web frameworks.

A distribution package is made runnable or hybrid by defining one or more `entry points` in the corresponding configuration file in `pyproject.toml` file or historically in `setup.py` or `setup.cfg`. The `pipx` tool will only let you install Python packages with at least one entry point. It will not work for runnable packages such as `Rich` or bare-bones libraries with Python code meant for importing.

As a side note in Python you can package up `.pyz` files which are kind of like `jar` files for java and you can have those files have all the dependencies you need to be able to distribute your Python project with all dependencies included.

You an install `pipx` via `pip`, but that will add a few dependencies to your global Python interpreter.  The command to do that would be `python -m pip install pipx`. You can also install it via your package manager.

## Configuring pipx before first use

You can use the command `pipx ensurepath` to add the path `pipx` uses (`/home/user/.local/bin`) to your `PATH` environment variable. You can also do this manually in your dot files.

To see what environment variables `pipx` has set you can use the command `pipx environment`.

Run the `pipx completions` command to get instructions for adding completion hints to your shell.

## Run Single-Use Python Apps

Lets say you want to try running Ruff the rust based python linter, but you don't want to set it up in your project.  You can just use the command `pipx run ruff check .`. With this command pipx will download the latest version of the requested package from PyPI and install it into a temporary virtual environment. It then will run the `ruff check .` command from that virtual environment without touching your project dependencies. To speed things up `pipx` will store the files in a cached location (`.` hidden directory) for two weeks. You can use the `--verbose` flag to see where this location is. You can also disable the cache with the `--no-cache` flag.

In the Ruff example the entry point and the name of the package are the same. If they are different or if the package has multiple entry points, you can use the `--spec` option. For example `pipx run --spec httpie http --body ifconfig.co/country`. This installs the HTTPie package and runs its `http` entry point to ge the message body of an HTTP response from `ifconfig.co/country` (this is a good one liner to check if your VPN is working correctly ifconfig.co/country is a service that checks for country of your http request).

You can also use the `--spec` option to run a specific version of a Python package. For example to run a specific version of poetry you would use the command `pipx run --spec 'poetry==1.1.11' poetry new your-project`. 

The requirement specifier has even more flexibility allowing you to bring extra dependencies or run a command straight from a remote Git repository or ZIP archive with the desired Python package.

`pipx run --spec git+https://github.com/realpython/reader.git realpython`

`pipx run --spec https://github.com/realpython/reader/archive/refs/tags/1.1.2.zip realpython`

The two commands above are an example of running the Real Python feed reader. The first command runs the `realpython` script directly from the default branch while the second one fetches the package from the giving release archive. You can optionally include a specific branch name or a commit hash in the Git URL by appending it after the `@` sign.

You can use `pipx` to run a Python script from any remote URL or local file as long as it has the `.py` extension.

```bash
ubuntu@ubuntu:~/python_practice/real_python/pipx_intro$ echo 'print("Hello, World!")' > hello.py
ubuntu@ubuntu:~/python_practice/real_python/pipx_intro$ pipx run hello.py
Hello, World!
```

If your script requires third party packages, then you can declare them in a specially formatted command, which must adhere to ***inline script metadata*** syntax from PEP 723 at the top of the file.

```python
# /// script
# dependencies = [
#   "rich==13.7.0",
# ]
# ///

from rich import print

print("[b]Hello, World[/b]")
```

When you run the above script through `pipx` it will create a temporary virtual environment and install the listed dependencies.

## Install Python Apps Globally

To install Ruff globally while keeping the systems interpreter intact you would use the command `pipx install ruff`. This creates an isolated virtual environment and install the latest version of the specified package into it. You can check the `pipx environment` to see where the permanent virtual environment is. `pipx` will also make a symbolic link for every entry point it finds in the installed package. This lets you run those tools directly by typing their names in your terminal as long as the directory is in your `PATH` variable.  For example is you install mypy with the command `pipx install mypy` you will get have globally available `dmypy, mypy, mypyc, stubgen, stubtest` as those are the entry points of the modules.

Some python packages such as `polars` don't define any entry points at all.  `pipx` will not install such packages and give you a meaningful error. If a dependency package has an entry point `pipx` will set that entry point.

The `pipx install` command does not have a `--spec` paramater like the `run` command, but you can use a version constraint for example `pipx install poetry==1.1.11`. 

Note that you can have at most one virtual environment per package because `pipx` names its virtual environments after the corresponding Python packages. To differentiate between versions of the same package you can provide a custom `--suffix` option (experimental feature). You can also overwrite an existing environment with a new version by using the `--force` flag.

## Manage Your Installed Apps

### List the Installed Apps

You don't have to memorize the individual Python entry points.  At any time you can run `pipx list` to see all of the virtual environments managed by `pipx` and their symbolic links and versions.  If you just want a list of packages without the environment details use `pipx list --short`.  To get full details of the underlying virtual environment of each package use `pipx list --json` which will give you a json output which may be handy for automated tasks.

### Upgrade Apps to Their Latest Versions

`pipx upgrade <package name>` for example `pipx upgrade ruff`

This command will use the correct virtual environment for the package and under the hood run `python -m pip install`. This saves you having to find and source the virtual environment yourself.

To upgrade all packages managed by `pipx` use `pipx upgrade-all`

### Downgrade Apps to a Specific Version

To downgrade or install a specific version you need to use the requirement specifier and the force flag to over write the existing installed version. For example `pipx install --force ruff=0.0.292` where `ruff` is the package you are downgrading. 

`pipx` has `reinstall-all` and `reinstall` commands they don't do what  you want in this scenario.

### Uninstall Apps and Virtual Environments

To remove a Python package managed by `pipx` and its corresponding virtual environment use the command `pipx uninstall ruff` where `ruff` is the package you wish to uninstall.

To uninstall everything use `pipx uninstall-all`

## Take Control Over Virtual Environments

### Inject Dependencies Into Managed Environments

If for example you installed Poetry using `pipx` and you want to add the optional plugin `poetry-plugin-export` you would use the `inject` functionality from `pipx`.  For example `pipx inject poetry poetry-plugin-export poetry-plugin-bundle`

The `inject` subcommand takes the original package name already installed and a list of extra dependencies.

To confirm that the dependencies were installed, you can use the `pipx runpip` command which lets you run arbitrary `pip` commands inside an apps virtual environment. The command would look similar to `pipx runpip poetry list | grep poetry`.

While you could use `runpip` to uninstall dependencies from a managed virtual environment, you're better off with a more concise `uninject` subcommand that pipx has to offer.  `pipx uninject poetry poetry-plugin-export`.

### Use a Specific Python Version in New Environments

By default, `pipx` will use the standard-library `venv` module from the Python interpreter you installed `pipx` with. You can override the default Python interpreter.

To override the interpreter before installing a package with `pipx` by modifying the `PIPX_DEFAULT_PYTHON` environment variable.  Both `pipx run` and `pipx install` will respect this variable.

```text
$ export PIPX_DEFAULT_PYTHON=/path/to/python-3.13.0a4/bin/python3
$ pipx run ipython
Python 3.13.0a4 (tags/v3.13.0a4:9d34f60783, Feb 26 2024, 13:03:08) [GCC 13.2.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.22.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]:
```

You can also use the `--python` argument. For example `pipx install --python=/home/user/.pyenv/versions/3.11.8/bin/python mypy`

If you want to change the python version in an existing environment you can use the `pipx reinstall` command. Run on its own it will remove the packages along with their virtual environment and install them again. You can combine this command with the `--python` parameter or setting the `PIPX_DEFAULT_PYTHON` environment variable to change the existing environment.

## Make Your Own APP for pipx

pick a Python packaging tool like setuptools, Flit, or Poetry, and specify your entry points accordingly. Then, build a wheel distribution package and publish it on PyPI. As a rule of thumb, you should first upload your package to TestPyPI for verification.