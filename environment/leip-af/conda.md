# Installing LEIP AF using a Python Conda Environment

This guide will walk through setting up a LEIP AF installation in a Conda environment. Using a Conda environment provides an isolated environment that will ensure that the dependencies required by AF are met and do not conflict with other Python packages in the same environment.

In order to use LEIP AF we need to ensure that our environment variables are properly configured as described in [the documentation](../README.md#workspace-configuration). If you are not already a user of Conda environments, you can follow these [instructions](https://docs.anaconda.com/free/miniconda/miniconda-install/) to get a minimal installation.

To begin, please make sure that Conda (or miniconda) is installed and available in your shell session:

```bash
conda list
```

Assuming everything is available, we can create a new environment and activate it inside our current shell session:

```bash
conda create -y -n af python=3.8
conda activate af
```

We can now continue to installing our packages with the comfort that everything is isolated inside our Conda environment. You can install Python packages in your virtual environment by following the steps for [installing from the repository](./README.md). Just remember to activate the `af` environment when working with LEIP tools!
