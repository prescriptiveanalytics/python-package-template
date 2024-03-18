# Poetry Cookiecutter

A simple [Cookiecutter](https://github.com/cookiecutter/cookiecutter) template for scaffolding Python packages and apps. The goal is to provide sane defaults for any application.

## Batteries Included

- **Packaging and dependency management** with [Poetry](https://github.com/python-poetry/poetry)
- **Ready to use CUDA DEVCONTAINER** with [devcontainers](https://containers.dev/)
- **Task running** with [Poe the Poet](https://github.com/nat-n/poethepoet)
   - **Pre defined** tasks
   - **Code formatting** with [Black](https://github.com/psf/black), [isort](https://github.com/PyCQA/isort)
   - **Code linting** with [Ruff](https://github.com/charliermarsh/ruff)
   - **Tests and test coverage** with [Pytest](https://github.com/pytest-dev/pytest/)
- **Scaffolding** updates with [Cookiecutter](https://github.com/cookiecutter/cookiecutter) and [Cruft](https://github.com/cruft/cruft)
- **Documentation generation** with [Mkdocs](https://github.com/mkdocs/mkdocs) and `mkdocstrings`
- Simple **Gitignore** for working with Python / PyCharm / VSCode / ...
- Default **EditorConfig** for Python and miscellaneous files
- Uses [Google-style docstrings](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings) or [NumPy-style](https://numpydoc.readthedocs.io/en/latest/format.html)
- Ready to use **Docker images** as a base CI/CD
- Cross-platform support for Linux, macOS (Apple silicon and Intel), and Windows

### Planned
- [ ] **Automatic dependency** updates with [RenovateBot]()

## Usage

### Creating a new Python project

1. Install [Cruft](https://github.com/cruft/cruft) and [Cookiecutter](https://github.com/cookiecutter/cookiecutter)
2. Run:
   ```sh
   cruft create -f https://github.com/prescriptiveanalytics/python-package-template
   ```
3. Install the environment with `poetry install` and create a container using `docker build -f Dockerfile-test .`
4. [Optional] Update your project template by running `cruft update`

## Parameters

| Parameter                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `package_name`                    <br> "My Awesome Proect"                         | The name of the package. Will be slugified to `snake_case` for importing and `kebab-case` for installing.                                                                                                                                                                                                                                                                                         |
| `package_description`             <br> "A single sentence description" | A single-line description of the package.                                                                                                                                                                                                                                                                                                                                                         |
| `package_url`                     <br> "https://gitdma.risc-software.at/<ID>/<package_name>" | The URL to the package's repository.                                                                                                                                                                                                                                                                                                                                                              |
| `author_name`                     <br> "Sonja Sunshine"                                 | The full name of the primary author of the package.                                                                                                                                                                                                                                                                                                                                               |
| `author_email`                    <br> "sonja.sunshine@risc-software.at"                           | The email address of the primary author of the package.                                                                                                                                                                                                                                                                                                                                           |
| `python_version`                  <br> "3.11"                                        | The minimum Python version that the package requires.                                                                                                                                                                                                                                                                                                                                             |
| `docstring_style`                 <br> ["Google", "Numpy"]                          | Whether to use and validate [NumPy-style](https://numpydoc.readthedocs.io/en/latest/format.html) or [Google-style docstrings](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings).                                                                                                                                                                                       |

## Supported Poe Tasks

- **precommit** Runs formatting, linting, and import sorting
- **check** Checks if formatting, linting and import sorting are corrcet
- **test** Runs all unit tests and doctests
- **docs** Serves documentation in the `docs` folder, also generates API docs from docstrings
