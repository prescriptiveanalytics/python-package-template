#TODO: might be cool to have a fixed image, to assure reproducibility
ARG FROM_IMAGE=debian:stable-slim
FROM FROM_IMAGE AS build

ARG PYTHON_VERSION={{cookiecutter.python_version}}
ARG POETRY_VERSION=1.8.0
ENV DEBIAN_FRONTEND=noninteractive

LABEL maintainer="{{cookiecutter.author_name}} <{{cookiecutter.author_email}}>"

RUN apt-get update && apt-get install -yq \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    curl \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev \
    git \
    python3-distutils \
    && rm -rf /var/lib/apt/lists/*
    
ENV PYENV_ROOT=$HOME/.pyenv
ENV PATH=$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH

# Install pyenv
RUN set -ex \
    && curl https://pyenv.run | bash \
    && pyenv update

# Install python and set interpreter
RUN pyenv install -s ${PYTHON_VERSION} \
    && pyenv global ${PYTHON_VERSION} \
    && pyenv rehash

# Install and configure Poetry
ENV POETRY_VERSION=${POETRY_VERSION} \
    POETRY_HOME=/opt/poetry \
    POETRY_VIRTUALENVS_CREATE=false
RUN curl -sSL https://install.python-poetry.org | python -
ENV PATH="${PATH}:${POETRY_HOME}/bin"

# Setup directory structure
RUN mkdir /workspace
WORKDIR /workspace

# set python path to default
ENV PYTHONPATH="${PYTHONPATH}:/workspace"

# Install dependencies (package itself is not installed)
COPY pyproject.toml poetry.lock ./
RUN poetry install --all-extras --no-root

# update any paths of installed cli tools (from pip)
RUN pyenv rehash

# Copy the rest of the repository
ADD . .


# TODO: build from FROM_IMAGE (or a plain python img), and just copy from build (without pyenv, poetry)
FROM build AS runtime


FROM base AS dev

RUN eval "$(pyenv init -)" \
    # activate pyenv on shell start
    && pyenv init - > ~/.bashrc

RUN poetry install --all-extras --no-root

RUN apt-get update --allow-insecure-repositories && apt-get install -yq \ 
    # provides nohup command
    coreutils \
    # editor
    vim \
    && rm -rf /var/lib/apt/lists/*