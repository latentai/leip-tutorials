## Setting Up LEIP using a Python Virtual Environment

These instructions will guide you through installing LEIP using a single docker container. In this environment, the [Compiler Framework](https://leipdocs.latentai.io/cf/latest/content/) (CF) will be launched as a server container. The [Application Framework](https://leipdocs.latentai.io/af/latest/content/) (AF) and a Juypter server are installed outside of a container in a Python virtual environment. A Python virtual environment will ensure that the dependencies required by AF are met and do not conflict with other Python packages you have installed on your system.  We will sometimes refer to the CF container as a server container because it contains a server API that enables calls to be made from outside of the container.

## Required Credentials

Begin by configuring the following environment variables as credentials: `LICENSE_KEY`, `REPOSITORY_TOKEN_NAME`, and `REPOSITORY_TOKEN_PASS`.


If you do not have a repository access token, please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the Help section. 

Create a directory as a shared volume for your Docker container:
 
```bash
mkdir ~/recipe_test
```

This allows a seamless file exchange between the Docker container and your host system. `The RECIPE_TEST_PATH` variable represents this directory.

```bash
export LICENSE_KEY=[your license key]
export RECIPE_TEST_PATH=~/recipe_test
```

If you have not already pulled the containers, log in to the Docker registry using your Personal Access Token.

```bash
docker login repository.latentai.com
username: <token-name>
password: <token-pass-code>
```

## Starting the Server Container

The following command will start the `leip_server` in a terminal window where the `LICENSE_KEY` has been set. This will start the server container (but not open a shell into it).

```bash
# Start the Compiler Framework docker container:
docker run -d -e=LICENSE_KEY \
       --gpus=all -p=8888:8888 \
       -v=$RECIPE_TEST_PATH:/recipe_test \
       --name=leip_server \
       repository.latentai.com/leip-cf:latest-gpu-cuda leip-server run
       
# Test the Compiler Framework docker container via HTTP, 
# should return {"detail":"Not Found"}
curl http://localhost:8888
```
## AF Installation

Now we will set up the Application Framework and Recipe Designer. Python virtual environments will be used to ensure the correct dependencies and to prevent dependency mismatches.

Begin by installing Python 3.8 and Python `venv` support: 

```bash
sudo apt install python3.8 python3.8-dev python3.8-venv
```

Next, create a Python virtual environment and activate it:

```bash
# For this example we'll choose the name af3 for our application 
# frameworks virtual environment.  We'll put it in our home directory:
mkdir ~/af3
cd ~/af3
python3.8 -m venv af3

# Activate the virtual environment:
source af3/bin/activate

# Set up your credentials:
export REPOSITORY_TOKEN_NAME=[your token name here]
export REPOSITORY_TOKEN_PASS=[your token secret here]
export PYPI_REPOSITORY_HOST=repository.latentai.com
export PYPI_REPOSITORY_PATH=/repository/pypi
export LEIP_PYPI_URL=https://${REPOSITORY_TOKEN_NAME}:${REPOSITORY_TOKEN_PASS}@${PYPI_REPOSITORY_HOST}${PYPI_REPOSITORY_PATH}/simple

# Now install the necessary packages:
pip install --upgrade pip
pip install -U pip wheel setuptools
pip install --extra-index-url $LEIP_PYPI_URL leip-af
pip install --extra-index-url $LEIP_PYPI_URL leip-recipe-designer-api
pip install --extra-index-url $LEIP_PYPI_URL leip-api
```

## Install and Run a Jupyter Notebook

Run the following commands to install and run a Juypter Notebook:

```bash
pip install jupyter
jupyter notebook --port 8889 --ip 0.0.0.0 --allow-root --no-browser &
```

Now verify that there are no conflicts:

```bash
pip check
```

You can use both `leip-recipe-designer` and `leip_client` APIs from the Python virtual environment. Remember to activate `AF3` when you want to use these APIs. You can also run a Jupyter Notebook on your host system by accessing [http://127.0.0.1:8889](http://127.0.0.1:8889)
