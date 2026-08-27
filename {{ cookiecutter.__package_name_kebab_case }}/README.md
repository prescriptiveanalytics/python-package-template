# {{ cookiecutter.package_name }}

{{ cookiecutter.package_description }}

## Overview

This project was generated from the RISC Python project template using [Cruft](https://github.com/cruft/cruft). It uses [uv](https://docs.astral.sh/uv/) for dependency management and [Poe the Poet](https://github.com/nat-n/poethepoet) for common development tasks.

## Requirements

* Python `{{ cookiecutter.python_version }}`
* [uv](https://docs.astral.sh/uv/)
* Docker, optional but recommended for containerized development and CI checks

## Setup

Clone the repository and change into the project directory:

```sh
git clone {{ cookiecutter.package_url }}
cd {{ cookiecutter.__package_name_kebab_case }}
```

Create or update the lock file:

```sh
uv lock
```

Install the project environment:

```sh
uv sync
```

## Usage

Run the package or application with:

```sh
uv run python -m {{ cookiecutter.__package_name_snake_case }}
```

Add project-specific usage examples here, for example CLI commands, Python API examples, or service startup instructions.

## Development

Common development tasks are defined with Poe and can be run through `uv`.

Format, sort imports, and lint:

```sh
uv run poe precommit
```

Check formatting, imports, linting, and licenses:

```sh
uv run poe check
```

Run tests:

```sh
uv run poe test
```

Serve the documentation locally:

```sh
uv run poe docs
```

Check dependency licenses:

```sh
uv run poe check_licenses
```

The license check exports dependencies to `requirements.txt` and writes a license report to `liccheck_reporting.txt`.

## Documentation

Documentation is built with [MkDocs](https://www.mkdocs.org/) and `mkdocstrings`.

Serve the documentation locally with:

```sh
uv run poe docs
```

Then open the local MkDocs URL printed in the terminal.

## Docker

This project includes Docker support for development, testing, and CI/CD workflows.

Build the test image with:

```sh
docker build -f Dockerfile-test .
```

Use the devcontainer setup for CUDA-enabled development, if applicable.

{% if cookiecutter.gpu_support %}
### Cuda Support

run `uv run poe check_cuda` to check your setup

{%- else %}
{%- endif %}


## CI/CD

The project includes a GitLab CI configuration in `.gitlab-ci.yml`.

Typical CI checks include:

```sh
uv sync --locked
uv run poe check
uv run poe test
```

Use `uv sync --locked` in CI to ensure the committed `uv.lock` file is up to date and reproducible.

## Updating from the Template

This project is connected to its original template through Cruft. Template metadata is stored in `.cruft.json`.

Check for template updates with:

```sh
cruft check
```

Update the project from the template with:

```sh
cruft update
```

After a template update, review the changes, resolve conflicts if needed, and rerun:

```sh
uv lock
uv sync
uv run poe check
uv run poe test
```

## Contributing

Before committing changes, run:

```sh
uv run poe precommit
uv run poe check
uv run poe test
```

Please keep the lock file up to date when dependencies change:

```sh
uv lock
```

## Repository

Repository: {{ cookiecutter.package_url }}
