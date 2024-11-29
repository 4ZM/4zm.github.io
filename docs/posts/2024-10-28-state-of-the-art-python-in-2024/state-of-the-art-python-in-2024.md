---
draft: false
date: 2024-10-28
slug: state-of-the-art-python-in-2024
categories:
  - Python
  - Best Practices
---

# State of the Art Python in 2024

Software development is about making choices. But available options change and so do the tradeoffs. Are you up to date with the best practices for creating a Python application in 2024? Let’s take a look at some great default choices.

<!-- more -->

<h3>TL;DR — State of the Art Python in 2024</h3>

1. Use **[uv](https://docs.astral.sh/uv/)** for dependency management (and everything else)
2. Use **[ruff](https://docs.astral.sh/ruff/)** for formatting and linting
3. Support Python 3.9 (or **3.13**)
4. Use a **pyproject.toml** file
5. Use **type hints**
6. Use **pytest** instead of unittest
7. Use **click** instead of argparse

## Use **[uv](https://docs.astral.sh/uv/)**

During 2024 the tool ‘**[uv](https://docs.astral.sh/uv/)**’ by Astral has taken the Python community by storm. Honestly, don’t know how we ever did without it.

uv is a bit like Rust’s [cargo](https://doc.rust-lang.org/cargo/), but for Python. It’s a Swiss army knife for working with your project. It does dependency management, handles your virtual environments, installs the right Python version, packages and more.

Instead of repeating all the guides in the official documentation, here are a few lovely things to try out:

```txt
uv init my-cli --app --package --python ">=3.9"
cd my-cli
uv python install "3.13"  # Install a python
uv run hello              # Run the 'main' func of the project
uv add click              # Add a dependency
uv add --dev pytest       # Add a development dependency
uv tool install .         # Install the app as a standalone application
uvx ruff format           # Format with ruff
```

I love [Poetry](https://python-poetry.org/), [pipx](https://pipx.pypa.io/stable/), [ruff](https://docs.astral.sh/ruff/), and [hatch](https://hatch.pypa.io/latest/) — and still use the latter two — but now [uv](https://docs.astral.sh/uv/) is the only front end you need to work with Python. It just works and it’s amazing ✨

NB: You should commit the `uv.lockfile` to get reproducible builds (and faster dependency resolution).

## Use **[ruff](https://docs.astral.sh/ruff/)**

The second amazing thing that Astral has created is the formatting and linting tool [ruff](https://docs.astral.sh/ruff/). It’s opinionated, very fast and you will never have to think about trivial formatting decisions ever again.

```txt
uvx ruff format
uvx ruff check --fix
```

It’s even fast enough to do formatting and auto-fixing in a git pre-commit hook! I suggest you run these two commands as a blocking PR check, and you will never have to discuss formatting in a code review ever again. And that… is AMAZING.

The only default I disagree with is the line width. In 2024, 120 is the new 80.

```toml
[tool.ruff]
line-length = 120
```

## Support Python 3.9 (or 3.13)

I have three guidelines for picking what Python version to use in 2024:

1. For public applications and libraries, you should support all of the [actively supported Python versions](https://devguide.python.org/versions/) out there. Right now, that is 3.9 to 3.13. This is a professional, grown-up (boring?) decision. Just do it.
2. For internal applications, where you are in control of the execution environment, use only the latest supported version. This leverages performance benefits, improves environment cohesion and gives you access to the latest and greatest features.
3. If you depend on a library that requires a more modern version than 3.9, be pragmatic about it. Either find a different library or accept limited reach for your own. Both are OK in different circumstances.

Using the latest Python version is a free lunch: your application and your tests are automatically faster and cheaper to run. Often, this translates to lower costs for your cloud services and CI infrastructure.

In practice you should add the default (3.13) version to a `.python-version` file in your project root. If you're nervous about 3.13 not having received it's first patch yet, staying on 3.12 for a few more weeks is fine.

You should also be explicit about supporting older versions in your `pyproject.toml` file:

```toml
[proj]
...
requires-python = ">=3.9"
```

## Use a pyproject.toml

Always use a `pyproject.toml` file to specify dependencies, build options, and tool configuration. This is your one stop shop for all things meta when it comes to your project.

This is sort of implied if you are using uv, so let’s not linger on it.

## Use Type Hints

Once a contentious issue, using type hints is now considered indispensable by most Python devs. In 2024, you should be using type annotations, especially in larger code bases.

Make sure you pick a tool for type checking that works well with your IDE and CI environment and move on. Some good options are [mypy](https://github.com/python/mypy), [Pyright](https://microsoft.github.io/pyright/), [pytype](https://google.github.io/pytype/), and [Pyre](https://pyre-check.org/).

## Use **[pytest](https://docs.pytest.org/en/stable/)**

Since you *are* writing unit tests (right?!) you need a test framework. The built-in [unittest](https://docs.python.org/3/library/unittest.html) is mostly around for legacy and compatibility in 2024. What you should be using is [pytest](https://docs.pytest.org/en/stable/).

Pytest has less boilerplate overhead for simple things (no required base class), and it brings more muscle for more complex tasks (parallel test execution & parameterized tests).

## Use **[click](https://click.palletsprojects.com/en/stable/)**

Command line interfaces are an absolute joy to develop in Python. Here are three libraries that you should use to step up your CLI game in 2024: [click](https://click.palletsprojects.com/en/stable/), [yaspin](https://github.com/pavdmyt/yaspin) and [tqdm](https://tqdm.github.io/).

[Click](https://click.palletsprojects.com/en/stable/) is a general CLI library. Its main feature is command line argument handling. It does parsing, shell completion and boilerplate generation. Compared to argparse there’s less to type and your code becomes more declarative (which is great).

```python
import click

@click.command()
@click.argument("url")
@click.option("--api", required=True, envvar="API_TOKEN", help="An API Token")
@click.option("--validation/--no-validation", is_flag=True, help="Perform validation?")
def main(url, api, validation):
    """A great utility"""
    pass
```

Gives us:

```text
Usage: my-cli [OPTIONS] URL

  A great utility

Options:
  --api TEXT                      An API Token  [required]
  --validation / --no-validation  Perform validation?
  --help                          Show this message and exit.
```

But click also handles other common command line task, like ANSI color and stdin/stdout “-” support for filename arguments.

```python
click.echo(click.style("✔", fg="green") + " Yaaaaz!")
```

Sometimes you need a bit more spice in your CLI. Use [yaspin](https://github.com/pavdmyt/yaspin) to add a terminal spinner and [tqdm](https://tqdm.github.io/) to add progress bars. Well isn’t that just lovely?

## Conclusion

It’s good to be a Python programmer in 2024. We have great libraries for all our basic needs like testing and CLI creation. By now, a consensus that type annotations are helpful has emerged, so you no longer need to argue with your coworkers about it. But more than anything, the tool uv is bringing a new level of convenience (and speed) to development workflows.

I would like to extend a huge **THANK YOU** to Astral and all other Python devs that work on the projects I‘ve mentioned. It’s never been more fun to write Python code, and it’s because of the work you do. 😍

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

**EDIT:** This topic caused quite a lot of discussion in my [post on Reddit](https://www.reddit.com/r/Python/comments/1ghiln0/state_of_the_art_python_in_2024). It has 600 up-votes! Check it out if you want to know what many other Python devs think about the subject.
