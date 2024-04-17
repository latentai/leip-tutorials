# Installing LEIP AF using a Docker Container

These instructions will guide you through installing LEIP AF as a Docker container. This guide also requires the LEIP CF, but it can be started by following the instructions inside the guide if it's not already running.

In this guide, the LEIP CF will be used via the Server API from a LEIP AF installation in a separate Docker container. The separate container provides an isolated environment that will ensure that the dependencies required by AF are met and do not conflict with other Python packages in the same environment.

## Authenticating to Docker Repository

In order to communicate with the Latent AI Docker Repository, you will need an access key pair. If you do not have a repository access token, please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the Help section. Once you have your key pair, you can use it to login to the Docker Repository:

```bash
docker login repository.latentai.com
```

```text
username: <token_name>
password: <token_value>
```

When your login is complete, you are ready to pull Docker images from Latent AI.

## Creating a LEIP AF Container

In order to create a LEIP AF container, please first ensure your environment variables are properly configured as described in [the documentation](../README.md#workspace-configuration). You can then use our [Docker Compose](https://docs.docker.com/compose/) definitions to launch your containers (from inside the [environment](../../environment/) directory).

As LEIP AF will generally be used alongside LEIP CF, we will start both containers using Docker Compose. If you are not planning on using LEIP CF (or if it's already running), feel free to exclude it from the list of activated services.

```bash
docker compose up leip-af leip-cf
```

With the containers started, you can access notebooks on your host system at [http://127.0.0.1:8888](http://127.0.0.1:8888). The default token to access the UI is `leip`, but it can be configured by setting `JUPYTER_TOKEN` before starting your containers.

If for some reason you need to run commands or Python code directly in one of the containers, you can do so by opening another terminal and attach to your container with:

```bash
docker exec -it leip-af bash
```

Or

```bash
docker exec -it leip-cf bash
```

## Troubleshooting

Error:
```
$ docker compose up leip-af leip-cf
service "leip-af" can't be used with `extends` as it declare `depends_on`
```
This is a known docker compose [issue](https://github.com/docker/compose/issues/11544) introduced in docker-compose v2.24.6 and fixed in a later version. Please upgrade your docker compose version. If installed via `apt-get`, run `sudo apt-get install --only-upgrade docker-compose-plugin`
