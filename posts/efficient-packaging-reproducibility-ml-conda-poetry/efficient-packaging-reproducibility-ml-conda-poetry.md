---
title: 'Efficient Packaging and Reproducibility for ML Projects with Conda and Poetry'
description: 'Discover how combining Conda and Poetry can streamline your machine learning project setup, ensuring easy packaging and reproducibility across environments.'
slug: 'efficient-packaging-reproducibility-ml-conda-poetry'
category: 
    - ai
    - python
published: '2024-02-21'
---

## Table of Contents

## Introduction

Machine Learning (ML) projects often require a delicate balance between flexibility in package management and the need for reproducibility across different environments. This guide introduces a robust approach combining Conda and Poetry, offering an efficient and reproducible packaging solution applicable not only to ML projects but to any Python-based project.

## Why Conda and Poetry?

Conda serves as a powerful package manager and environment manager. It simplifies package installation and environment management across various platforms. Poetry, on the other hand, excels in dependency management and packaging of Python projects. By leveraging the strengths of both, developers can manage dependencies more effectively and ensure that projects are easily reproducible.

## Setting Up a Reproducible Environment

### Initial Setup with Conda

Create an `environment.yml` file specifying Conda, Python version, Poetry, Mamba, and essential packages. This file serves as the foundation for creating a consistent environment across different setups.

```yaml:environment.yml showLineNumbers
name: my_project_env
channels:
  - pytorch
  - nvidia
  - conda-forge
  - nodefaults
dependencies:
  - python=3.11
  - mamba  # Use mamba for faster dependency resolution
  - conda-lock
  - pip
  - poetry=1.*
  - pytorch::pytorch=2.2.0
  - pytorch::torchaudio=2.2.0
  - pytorch::torchvision=0.17.0
  - pytorch::pytorch-cuda=12.1

# Non-standard section listing target platforms for conda-lock:
platforms:
  - linux-64
```

`virtual-packages.yml`` (may be used e.g. when we want conda-lock to generate CUDA-enabled lock files even on platforms without CUDA):

```yaml:virtual-packages.yml showLineNumbers
subdirs:
  linux-64:
    packages:
      __cuda: 12.1
```

### Enhancing Reproducibility with `conda-lock`

`conda-lock` generates lock files for Conda dependencies, akin to `poetry.lock` for Poetry dependencies. This ensures that you can recreate the exact environment elsewhere.

### Using Mamba for Dependency Resolution

Mamba is a reimplementation of the Conda package manager in C++. It offers faster and more reliable dependency resolution. This is particularly useful when dealing with complex dependency trees in ML projects.

### Dependency Management with Poetry

By default, use Poetry to add Python dependencies. If a package is not available on PyPI or is better installed via Conda (e.g., for a CUDA-enabled version), specify it in `environment.yml`. Ensure to sync the version in `pyproject.toml` to prevent unintended upgrades.

### Handling Channels and Package Versions

When using multiple channels that provide the same packages, it's crucial to specify the channel using the `::` notation or enable strict channel priority to ensure reproducibility. [Conda's strict channel priority](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html#strict).

### Avoiding Non-Reproducibility Due to User Site-Packages

Ensure the `PYTHONNOUSERSITE` environment variable is set to `True` to prevent Python from adding user site-packages to `sys.path`, which can lead to non-reproducibility. [User site-packages](https://docs.python.org/3/library/site.html#site.USER_SITE) [PYTHONNOUSERSITE](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONNOUSERSITE)

## First-Time Setup Steps

1. **Create the Conda environment** from the `environment.yml` file.
2. **Generate Conda lock files** using `conda-lock`.
3. **Set up Poetry** ensuring the Python version matches.
4. **Fix package versions** installed by Conda in `pyproject.toml`.
5. **Add Conda and Poetry spec and lock files** to version control.

```shell:terminal
# Create the Conda environment
conda env create -f environment.yml
# Create Conda lock file(s) from environment.yml
conda-lock -k explicit --conda mamba
# Set up Poetry
poetry init --python=~3.11  # version spec should match the one from environment.yml
# Fix package versions installed by Conda to prevent upgrades
poetry add --lock torch=2.2.0 torchaudio=2.2.0 torchvision=0.17.0
# Add conda-lock (and other packages, as needed) to pyproject.toml and poetry.lock
poetry add --lock conda-lock
```

Optinally store/commit the lock files:

```shell:terminal
# Add Conda spec and lock files
git add environment.yml virtual-packages.yml conda-linux-64.lock
# Add Poetry spec and lock files
git add pyproject.toml poetry.lock
git commit
```

## Reproducing the Development Environment

Now you have the local dev envs setup. Now to reproduce your dev env for containers or pulishing you only need to do 3 commands🔥:

```shell:terminal
# Create the Conda environment from the lock file
conda create --name my_project_env --file conda-linux-64.lock
conda activate my_project_env
# Install Poetry dependencies
poetry install
```

## Updating the environment

If you need to update the environment, you can do so by updating the `environment.yml` file and re-generating the lock files. Then, update the Conda packages and Poetry dependencies based on the re-generated lock files.

```shell:terminal
# Re-generate Conda lock file(s) based on environment.yml
conda-lock -k explicit --conda mamba
# Update Conda packages based on re-generated lock file
mamba update --file conda-linux-64.lock
# Update Poetry packages and re-generate poetry.lock
poetry update
```
