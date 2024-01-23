This section guides you through the process of setting up an environment that can concurrently access the [Application Framework](https://leipdocs.latentai.io/af/latest/content/) and [Compiler Framework](https://leipdocs.latentai.io/cf/latest/content/) using containers. To complete this procedure, please ensure that [Docker](https://docs.docker.com/engine/install/):octicons-link-external-16: is installed on your system.

## Required Credentials

First, configure the following environment variables as credentials: `LICENSE_KEY`, `REPOSITORY_TOKEN_NAME`, and `REPOSITORY_TOKEN_PASS`.


!!! tip "If you don't have a repository access token"
    Please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the help section. 

Create a directory as a shared volume for your Docker container:
 
```bash
mkdir recipe_test
```

This allows seamless file exchange between the Docker container and your host system. `The RECIPE_TEST_PATH` variable represents this directory

```bash
export LICENSE_KEY=[your license key]
export RECIPE_TEST_PATH=/home/username/recipe_test
```

If you haven't already pulled the containers, login to the docker registry using your Personal Access Token

```bash
docker login repository.latentai.com
username: <token-name>
password: <token-pass-code>
```

## Server Container

The following command will start the `leip-server` in a terminal window where the `LICENSE_KEY` has been set. This will start the server container (but not open a shell into it).

```bash
docker run -d -e=LICENSE_KEY \
       --gpus=all -p=8888:8888 \
       -v=$RECIPE_TEST_PATH:/recipe_test \
       --name=leip_server \
       repository.latentai.com/leip-cf:latest-gpu-cuda leip-server run
```

## Client Container

The following command will start the generic Python 3.8 container we will use to construct the client-side environment:
```bash
docker run -e=LICENSE_KEY --name=recipe_test \
       -it -p=8889:8889 --ipc=host --gpus=all \
       --add-host=host.docker.internal:host-gateway \
       -v=$RECIPE_TEST_PATH:/recipe_test python:3.8 bash
```
!!! tip
    If you exit the container and want to reattach to it, use `docker start recipe_test; docker attach recipe_test `


Once you are inside the container run the following:
```dockerfile
export REPOSITORY_TOKEN_NAME=[your token name here]
export REPOSITORY_TOKEN_PASS=[your token secret here]
export PYPI_REPOSITORY_HOST=repository.latentai.com
export PYPI_REPOSITORY_PATH=/repository/pypi
export LEIP_PYPI_URL=https://${REPOSITORY_TOKEN_NAME}:${REPOSITORY_TOKEN_PASS}@${PYPI_REPOSITORY_HOST}${PYPI_REPOSITORY_PATH}/simple
pip install --upgrade pip
pip install --extra-index-url $LEIP_PYPI_URL leip-af
pip install --extra-index-url $LEIP_PYPI_URL leip-recipe-designer-api
pip install --extra-index-url $LEIP_PYPI_URL leip-api

# FIXME, these are currently needed to avoid some errors
pip install fiftyone
pip install fiftyone-db-ubuntu2204

# FIXME, this needs to be installed for now to avoid an issue in AF train
pip install tensorboard

# Install and run jupyter notebook
pip install jupyter
jupyter notebook --port 8889 --ip 0.0.0.0 --allow-root --no-browser &

# test that you can access the server container via HTTP, should return {"detail":"Not Found"}
curl http://host.docker.internal:8888
```

In this client container environment, you can use both `leip-recipe-designer` and `leip_client` APIs from Python. You can also run a Jupyter notebook accessing port 8889 on the host system.

