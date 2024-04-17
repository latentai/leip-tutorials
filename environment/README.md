# Setting up a LEIP Environment

To use the full LEIP platform, you will need to install
the [Application Framework](https://leipdocs.latentai.io/af/latest/content/) (AF) and
the [Compiler Framework](https://leipdocs.latentai.io/cf/latest/content/) (CF). Each tool can be used independently, but
you may need any number of them based on your use case. The LEIP CF offers a Web API component which allows
communication from client libraries installed in other environments. This means that you can use a different
installation for other LEIP tools easily.

## Minimum System Requirements

### Local

| Hardware | Operating System | RAM  | GPU    |
|----------|------------------|------|--------|
| x86_64   | Ubuntu 20 or 22  | 16GB | NVIDIA |

### Cloud

| Provider | Hardware Architecture | AMI                                   | RAM  | GPU    |
|----------|-----------------------|---------------------------------------|------|--------|
| AWS      | g4dn.xlarge           | NVIDIA deep learning, Ubuntu 20 or 22 | 16GB | NVIDIA |

## Configuration

In order to use any of the LEIP tooling effectively, you must designate both a LEIP License Key and a workspace. This
can be done by setting the `LICENSE_KEY` and `LEIP_WORKSPACE` environment variables in your shell session:

```bash
export LICENSE_KEY=key/<license_key>
export LEIP_WORKSPACE=/path/to/your/workspace
```

These values will be used inside notebooks and throughout examples in order to write to the correct locations. If you
are planning on using more than one LEIP tool in combination with another, please ensure these values match across your
shell environments.

## Installation

Please ensure that [Docker](https://docs.docker.com/engine/install/) is installed on your system prior to attempting to
install any tools using Docker containers. We recommend you run on a machine equipped with an NVIDIA GPU. To access the
GPU from the LEIP Docker containers, you will also need to install
the [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
.

### LEIP AF

* [Installation from the PyPI Repository](./leip-af/README.md)
* [Installation inside a Docker container](./leip-af/docker.md)
* [Installation inside a Conda environment](./leip-af/conda.md)
* [Installation inside a Python virtual environment](./leip-af/venv.md)

### LEIP CF

* [Installation inside a Docker container](./leip-cf/docker.md)
