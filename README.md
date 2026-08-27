# Python Project Template

A [Cruft](https://github.com/cruft/cruft)-managed template for scaffolding Python packages and applications. The goal is to provide sane defaults for new Python projects.

## Usage

### Creating a new Python project

Docker is the only required dependency for creating a new project from this template. `cruft` is executed inside a temporary Docker container using `uvx`.

1. `cd` to the desired parent directory of your new project.

2. Run the project generator.

   **macOS / Linux / WSL:**

   ```sh
   docker run --rm -it \
     --user "$(id -u):$(id -g)" \
     -e HOME=/tmp \
     -v "$PWD:/work" \
     -w /work \
     ghcr.io/astral-sh/uv:debian \
     uvx cruft create -f https://gitdma.risc-software.at/common/python-package-template
   ```

   **Windows PowerShell:**

   ```powershell
   docker run --rm -it `
     -e HOME=/tmp `
     -v "${PWD}:/work" `
     -w /work `
     ghcr.io/astral-sh/uv:debian `
     uvx cruft create -f https://gitdma.risc-software.at/common/python-package-template
   ```
   This starts the interactive project creation prompt and creates the directory for your new project.

   > FYI: Optional: test a specific template branch by adding `--checkout <BRANCH_NAME>`.
   > FYI: For errors relating to the docker credential store see the [FAQ section](#FAQ).

3. After project creation, you can edit `<your_project>/.cruft.json` if you need to adjust the recorded template inputs.

4. Change into the generated project directory:

   ```sh
   cd <your_project>
   ```

5. Generate or update the lock file:

   ```sh
   uv lock
   ```

6. Install the project environment:

   ```sh
   uv sync
   ```

7. Optional: update your project from the template later by running:

   ```sh
   cruft update
   ```

## Batteries Included

* **Packaging and dependency management** with [uv](https://docs.astral.sh/uv/)

* **Ready-to-use CUDA devcontainer** with [Dev Containers](https://containers.dev/)

* **Task running** with [Poe the Poet](https://github.com/nat-n/poethepoet)

  * **Predefined tasks**
  * **Code formatting** with [Black](https://github.com/psf/black) and [isort](https://github.com/PyCQA/isort)
  * **Code linting** with [Ruff](https://github.com/astral-sh/ruff)
  * **Tests and test coverage** with [pytest](https://github.com/pytest-dev/pytest/)

* **Scaffolding updates** with [Cookiecutter](https://github.com/cookiecutter/cookiecutter) and [Cruft](https://github.com/cruft/cruft)

* **Documentation generation** with [MkDocs](https://github.com/mkdocs/mkdocs) and `mkdocstrings`

* Simple **`.gitignore`** for Python, PyCharm, VS Code, and related tooling

* Default **EditorConfig** for Python and miscellaneous files

* Support for [Google-style docstrings](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings) and [NumPy-style docstrings](https://numpydoc.readthedocs.io/en/latest/format.html)

* Ready-to-use **Docker images** for CI/CD

* Cross-platform support for Linux, macOS, and Windows

### Planned

* [ ] **Automatic dependency updates** with Renovate

## Parameters

| Parameter             | Default                                                 | Description                                                                                                                                     |
| --------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `package_name`        | `"My Awesome Project"`                                  | The name of the package. It will be slugified to `snake_case` for importing and `kebab-case` for installing.                                    |
| `package_description` | `"A single sentence description"`                       | A single-line description of the package.                                                                                                       |
| `package_url`         | `"https://gitdma.risc-software.at/<ID>/<package_name>"` | The URL of the package repository.                                                                                                              |
| `author_name`         | `"Sonja Sunshine"`                                      | The full name of the primary author of the package.                                                                                             |
| `author_email`        | `"sonja.sunshine@risc-software.at"`                     | The email address of the primary author of the package.                                                                                         |
| `python_version`      | `"3.12"`                                                | The target Python minor version for the project, for example `3.12`. The generated project requires this Python minor version, e.g. `~=3.12.0`. |
| `docstring_style`     | `["Google", "Numpy"]`                                   | Whether to use and validate Google-style or NumPy-style docstrings.                                                                             |

## Supported Poe Tasks

* **precommit**: Runs formatting, linting, and import sorting.
* **check**: Checks whether formatting, linting, and import sorting are correct.
* **test**: Runs all unit tests and doctests.
* **docs**: Serves the documentation in the `docs` folder and generates API documentation from docstrings.
* **check_licenses**: Checks whether dependency licenses are allowed or need to be reviewed.

## FAQ

  <details>
  <summary>Docker credential error on windows</summary>
  
  > If you get an `docker: error getting credentials - err: exit status 1, out: A specified logon session does not exist.` error you can work around it by using this commmand:

  ```powershell
  mkdir -p /tmp/docker-empty-config

  DOCKER_CONFIG=/tmp/docker-empty-config docker run --rm -it \
    --user "$(id -u):$(id -g)" \
    -e HOME=/tmp \
    -v "$PWD:/work" \
    -w /work \
    ghcr.io/astral-sh/uv:debian \
    uvx cruft create -f --checkout main https://gitdma.risc-software.at/common/python-package-template
  ```
  </details>   
