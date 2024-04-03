# Installing LEIP AF using a Docker Container

These instructions will guide you through installing LEIP AF as a Docker container. This guide also requires the LEIP CF, but it can be started by following the instructions inside guide if it's not already running.

In this guide, the LEIP CF will be used via the Server API from a LEIP AF installation in a separate Docker container. The separate container provides an isolated environment that will ensure that the dependencies required by AF are met and do not conflict with other Python packages in the same environment.

## Pulling the LEIP AF Image

In order to communicate with the Latent AI Docker Repository, you will need an access key pair. If you do not have a repository access token, please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the Help section. Once you have your key pair, you can use it to login to the Docker Repository:

```bash
$ docker login repository.latentai.com
username: <token_name>
password: <token_value>
```

When your login is complete, you can pull the latest version of the LEIP AF container to check that everything worked as expected.

```bash
$ docker pull repository.latentai.com/leip-af:latest
```

If you're planning on also using the LEIP CF alongside LEIP AF, you can also choose to [pull that image](../leip-cf/docker.md#pulling-the-leip-cf-image) at this point too. Please note that this may take a while, so maybe get a head start on installing any other LEIP tools you need in the meantime!

## Creating a LEIP AF Container

In order to create a LEIP AF container, please first ensure your environment variables are properly configured as described in [the documentation](../README.md#workspace-configuration). You can then use our [Docker Compose](https://docs.docker.com/compose/) definitions to launch your containers (from inside the [environment](../../environment/) directory):

```bash
$ docker compose up leip-af leip-cf
```

As LEIP AF will generally be used alongside LEIP CF, we will start both containers using Compose. If you're not planning on using LEIP CF (or if it's already running), feel free to exclude it from the list of activated services. If you need to attach to your running compose container, you can do so:

```bash
$ docker exec -it leip-af bash
```

From here you can run any Python code or programs you need, or install any required Python packages. By default you will be placed inside your configured workspace directory (via `$LEIP_WORKSPACE`).

With the containers started, you can access notebooks on your host system at [http://127.0.0.1:8888](http://127.0.0.1:8888). The default token to access the UI is `leip`, but can be configured by setting `JUPYTER_TOKEN` before starting your containers.
