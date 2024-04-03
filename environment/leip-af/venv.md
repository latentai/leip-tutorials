# Installing LEIP AF using a Python Virtual Environment

This guide will walk through setting up a LEIP AF installation in a Python virtual environment. The Python virtual environment provides an isolated environment that will ensure that the dependencies required by AF are met and do not conflict with other Python packages in the same environment.

In order to use LEIP AF we need to ensure that our environment variables are properly configured as described in [the documentation](../README.md#workspace-configuration). Please also make sure that you are using Python 3.8 and have support for virtual environments. On Debian-based systems, this can be done by installing the following packages:

```bash
apt install python3.8 python3.8-dev python3.8-venv
```

Once installed we can create a new workspace for our virtual environment:

```bash
mkdir -p ~/leip/af
cd ~/leip/af
```

We can then activate it inside our current shell session:

```bash
python3.8 -m venv af
source ~/leip/af/bin/activate
```

We can now continue to installing our packages with the comfort that everything is isolated inside our virtual environment. You can install Python packages in your virtual environment by following the steps for [installing from the repository](./README.md). Just remember to activate the `af` environment when working with LEIP tools!
