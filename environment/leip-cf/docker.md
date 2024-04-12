# Installing LEIP CF using a Docker Container

These instructions will guide you through installing LEIP CF as a Docker container. If you are planning on using LEIP CF alongside LEIP AF, you should follow [the LEIP AF guide](../leip-af/docker.md) instead.

## Pulling the LEIP CF Image

In order to communicate with the Latent AI Docker Repository, you will need an access key pair. If you do not have a repository access token, please refer to the topic ["How do I create a Personal Access Token?"](https://leipdocs.latentai.io/home/content/help/#installing-leip) in the Help section. Once you have your key pair, you can use it to login to the Docker Repository:

```bash
docker login repository.latentai.com
```

```text
username: <token_name>
password: <token_value>
```

When your login is complete, you are ready to pull Docker images from Latent AI.

## Creating a LEIP CF Container

In order to create a LEIP CF container, please first ensure your environment variables are properly configured as described in [the documentation](../README.md#workspace-configuration). You can then use our [Docker Compose](https://docs.docker.com/compose/) definitions to launch your containers (from inside the [environment](../../environment/) directory):

```bash
docker compose up leip-cf
```

If for some reason you need to run commands or Python code directly in the CF container, you can do so by opening another terminal and attach to your container with:

```bash
docker exec -it leip-cf bash
```

From here you can run any Python code or programs you need, or install any required Python packages. By default, you will be placed inside your configured workspace directory (via `$LEIP_WORKSPACE`).
