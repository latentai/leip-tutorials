## Installing LEIP

To use the full LEIP, you will need to install the [Application Framework](https://leipdocs.latentai.io/af/latest/content/) and the [Compiler Framework](https://leipdocs.latentai.io/cf/latest/content/). Application Framework (AF) can be installed using a Docker container or using a Python virtual environment. Compiler Framework (CF) installation is currently only supported within a separate Docker container. The CF container contains a server component that allows a client API installed in either the AF Docker container or the AF Python virtual environment to communicate with the CF server.  Choose the appropriate installation path that meets your needs.

* [Installing LEIP using AF in a Docker container](./client_docker.md)
* [Installing LEIP using AF in a Python virtual environment](./python_virtual.md)

Please ensure that [Docker](https://docs.docker.com/engine/install/) is installed on your system prior to attempting to install Application Frameworks using either methods.  We recommend you run on a machine equipped with an NVIDIA GPU. To access the GPU from the LEIP containers, you will also need to install the [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).







