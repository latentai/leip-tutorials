# Installing LEIP AF Python Packages

This guide will cover installing `leip-af` and `leip-recipe-designer-api` Python packages from the Latent AI PyPI Repository, which are need to use LEIP AF.

## Required Credentials

In order to communicate with the LatentAI PyPI Repository, you will need an access key pair. If you do not have a repository access token, please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the Help section. Once you have your personal token, place it inside your shell session so it can be referred to by further instructions:

```bash
$ REPOSITORY_TOKEN_NAME=<token_name>
$ REPOSITORY_TOKEN_PASS=<token_value>
```

You may also find it helpful to define the following values if you're planning on installing other packages in the future:

```bash
$ REPOSITORY_HOST=repository.latentai.com
$ REPOSITORY_PATH=/repository/pypi
$ REPOSITORY_URL=https://${REPOSITORY_TOKEN_NAME}:${REPOSITORY_TOKEN_PASS}@${REPOSITORY_HOST}${REPOSITORY_PATH}/simple
```

With these values configured, you are now ready to install packages from the repository.

## Installing Packages

We can now use the repository to install `leip-af`, as well as `leip-recipe-designer-api`. If you are planning on communicating with LEIP CF (as is done in the [Getting Started](../../notebooks/GettingStarted.ipynb) notebook) you may also install `leip-api`.

```bash
$ pip install --upgrade pip wheel setuptools
$ pip install --extra-index-url $REPOSITORY_URL leip-api leip-af leip-recipe-designer-api
```

This will install all packages you need to use LEIP AF, as well as packages needed to communicate with the LEIP CF Server.

## Running a Jupyter Notebook

You can now also install Jupyter in order to run notebooks using LEIP AF, should you wish to. You can do this by installing the required packages:

```bash
$ pip install jupyter
$ pip check
```

You can then run the Jupyter server to enable working alongside the notebook tutorials and examples:

```bash
$ jupyter notebook --port 8889 --ip 0.0.0.0 --allow-root --no-browser
```

You can access these notebooks on your host system at [http://127.0.0.1:8889](http://127.0.0.1:8889).
