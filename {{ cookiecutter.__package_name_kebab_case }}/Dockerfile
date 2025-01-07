#TODO: might be cool to have a fixed image, to assure reproducibility
ARG FROM_IMAGE=python:{{cookiecutter.python_version}}
ARG FROM_IMAGE_RUNTIME=build
ARG FROM_IMAGE_DEV=build

FROM $FROM_IMAGE AS build

ARG POETRY_VERSION=1.8.3
ENV DEBIAN_FRONTEND=noninteractive

LABEL maintainer="{{cookiecutter.author_name}} <{{cookiecutter.author_email}}>"

# Here you can add any system dependencies you might need
RUN apt-get update && apt-get install -yq \
    # useful for installing python tools
    pipx \
    # cleanup
    && apt-get clean && rm -rf /var/lib/apt/lists/*
    
# Install and configure the poetry
ENV PATH="/root/.local/bin:${PATH}" \
    POETRY_VERSION=${POETRY_VERSION} \
    POETRY_VIRTUALENVS_IN_PROJECT=false \
    POETRY_CACHE_DIR=/tmp/poetry_cache
RUN pipx install poetry==${POETRY_VERSION} && pipx ensurepath

# Setup directory structure
# https://github.com/GoogleContainerTools/kaniko/issues/1508
# seems like remnant of an earlier kaniko implementation
# delete, to circumvent any side-effects
# can be removed after issue above is fixed
RUN rm -rf /workspace

RUN mkdir /workspace
WORKDIR /workspace

# Install dependencies (package itself is not installed)
COPY pyproject.toml *.lock ./

# MOD: Add private PYPI repository with credentials - ensure the build args are set!
# poetry config http-basic.<private-pypi> ${POETRY_AUTH_POWERCAST_USERNAME} ${POETRY_AUTH_POWERCAST_PASSWORD}

RUN poetry install --without dev --all-extras --no-root && rm -rf $POETRY_CACHE_DIR{artifacts,cache}

###############################################################################
# TODO: build from FROM_IMAGE (or a plain python img), and just copy from build (without pyenv, poetry)
FROM $FROM_IMAGE_RUNTIME AS runtime

# Copy the rest of the repository
COPY . .

CMD ["bash"]

###############################################################################
FROM $FROM_IMAGE_DEV AS dev

# Point python path to workspace
ENV PYTHONPATH="${PYTHONPATH}:/workspace"

# Install dev dependencies
RUN poetry install --all-extras --no-root && rm -rf $POETRY_CACHE_DIR{artifacts,cache}

# Install dev tools conditionally 
ARG INSTALL_DEV_TOOLS=false
RUN if [ "$INSTALL_DEV_TOOLS" ]; then \
    apt-get update && apt-get install -yq \
    coreutils \
    neovim \
    curl \
    iputils-ping \
    && apt-get clean && rm -rf /var/lib/apt/lists/*; \
    fi