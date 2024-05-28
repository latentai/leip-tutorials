# Setting up LEIP on an AWS EC2 Instance

This guide will walk you through starting an Amazon EC2 instance and installing
the LEIP tools. At the end of this tutorial you will have an Amazon EC2 instance
with support for compiling and optimizing models, with remote access to Jupyter
on the machine for running notebooks.

## Requirements

Following this tutorial requires the following:

* A valid LEIP product license key
* An Amazon AWS account, with some knowledge of Amazon EC2
* Credentials to access the LEIP Docker repository
* Some knowledge of Docker and Docker Compose

This guide should generally take you around 30 minutes, although following any
optional steps may take a few minutes longer.

## Step One: Creating your Amazon Instance

We'll begin by logging into AWS EC2 and creating a new instance. From the AWS
EC2 console, select "Launch Instances" from the top right of the screen. This
will drop you into a form to create your instance. You can use the following
as a guide for filling out this form:

1. You can name the instance whatever you want
2. Under "Application and OS Images" we recommend to use a Quick Start AMI
    * Select "Ubuntu" in the Quick Start menu
    * Expand the dropdown and search for "Deep Learning Base OSS Nvidia"
    * There should be a single result, so select this as your base image
    * Please see the [release notes](https://aws.amazon.com/releasenotes/aws-deep-learning-base-gpu-ami-ubuntu-22-04/)
      for the list of supported instance types
3. Under "Instance Type" you can select your instance type and size
    * If you don't have specific requirements, you should select the
      `g4dn.xlarge` instance type.
    * If picking your own instance type, please make sure it's supported
      by checking the notes in the link above.
4. Under "Key Pair" you can select/create a key pair for your instance
    * Although you can skip this step, it is highly recommended to secure
      access to your instance by assigning a key pair.
    * This may not be a concern if you only allow SSH from specific IPs.
5. Under "Network Settings", make sure to click "Edit" to open the advanced
   configuration settings.
    * Make sure that "Auto-assign public IP" is set to "Enable"
    * Create a new security group named "leip-sg"
    * There are several recommended rules to add to this group:
        * Allow `SSH` on port `22` from either your IP or anywhere (if using a
          key pair).
        * Allow `Custom TCP` on port `8888` if you wish to access Jupyter and
          use Jupyter notebooks directly from your workstation
        * Allow `Custom TCP` on port `8080` if you wish to access LEIP CF via
          the LEIP Server from your workstation
6. Under "Configure Storage" choose your volume size
    * We recommend starting with at least `256gb` on a `gp3` volume
    * If you anticipate you'll need more, feel free to raise this higher now to
      save yourself the trouble of having to scale up later

This may seem like a lot of steps, but it should only take you a couple of minutes
to run through the form before clicking "Launch Instance". You should be taken to
a new screen where you can see something like the following:

```
Successfully initiated launch of instance (i-*****************)
```

You can click on the `i-*` tag to be taken to your instance in AWS EC2. If you
select your instance, you'll be able to see `Public IPv4 address` in the details
tab at the bottom. You can now use this to login to your instance:

```bash
ssh -i <path/to/your/key> ubuntu@<your-ip>
```

Once you're inside the system, you can continue on to the next step. Please note
that it might take a couple of minutes for the instance to fully start and your
login to work successfully.

## Step Two: Configuring your Environment

Once you're logged into your machine, the first step is to make sure everything
is up to date. This is important as it will update Nvidia drivers, as well as the
Nvidia Docker toolkit, to the the latest versions.

```bash
sudo apt update
sudo apt upgrade -y
```

This may take a little while, but afterwards you should have the latest versions
of everything so you can be sure you're up to date.

There are several variables we'll use on this instance to configure the LEIP
tooling. You can set these inside `~/.bashrc` to avoid having to provide them
every time you login.

```bash
export LICENSE_KEY=key/***
export JUPYTER_TOKEN=<token>
export LEIP_DIRECTORY=/opt/latentai
export LEIP_WORKSPACE=$LEIP_DIRECTORY/workspace
```

The `LICENSE_KEY` should match the one you have been provided by Latent AI, as it's
used by the tools to validate your usage. The `JUPYTER_TOKEN` value is optional, but
allows you to specify a token required to access your Jupyter installation. It's
recommended that you put something secure here, in order to stop others from being
able to access your Jupyter installation.

You can also take this opportunity to save yourself headaches later by configuring
your LEIP Repository credentials:

```bash
export LEIP_REPOSITORY_USERNAME=<user>
export LEIP_REPOSITORY_PASSWORD=<pass>
export PIP_EXTRA_INDEX_URL=https://$LEIP_REPOSITORY_USERNAME:$LEIP_REPOSITORY_PASSWORD@repository.latentai.com/repository/pypi/simple
```

Please note that you will need to either `source ~/.bashrc` or log out/in to refresh
your current shell variables.

## Step Three: Installing the LEIP Repositories

LEIP comes with several tutorials and example applications which can be used as a
reference when producing your own applications. In order to get up and running
quickly, we can use these tools as a launchpad by installing them in our workspace:

```bash
sudo mkdir -p $LEIP_WORKSPACE
sudo git clone https://github.com/latentai/leip-tutorials $LEIP_DIRECTORY/tutorials
sudo git clone https://github.com/latentai/example-applications $LEIP_DIRECTORY/example-applications
sudo chown -R ubuntu:ubuntu $LEIP_DIRECTORY
```

The tutorials repository also contains the definitions we'll use to launch our Docker
containers, while the example applications repository includes reference applications
across a variety of languages which you can use to run your LEIP artifacts.

Please note that you can set up your workspace and repositories in whichever locations
you wish, but the above are the recommended locations to keep everything organized.

## Step Four (Optional): Installing Application Dependencies

_If you're not planning on running LEIP example applications on your AWS instance, you
can safely skip this section and continue to the next step._

In order to build and run example applications, the first step is to install `liblre`
on your system. This is the core runtime engine for LEIP artifacts, and will be used
when executing your models.

To do this we need to add the Latent AI `apt` repository to our Ubuntu installation,
so we can install the necessary `liblre` libraries along with any dependencies they
might have:

```bash
curl https://public.latentai.io/add_apt_repository | sudo bash
sudo apt update
sudo apt install -y liblre-cuda12 liblre-dev
sudo apt install -y libnvinfer-dev=8.6.1.6-1+cuda12.0 libnvinfer-headers-dev=8.6.1.6-1+cuda12.0
```

With these libraries installed, we can now move on to choosing an example application
we'd like to execute our models. For this guide the Python example application gives us
an excellent starting point as it provides a very smooth installation process via `pip`:

```bash
python3 -m pip install pylre
```

You will then also need some additional dependencies based on which type of application
you're planning on running:

```bash
# Classifiers
python3 -m pip install albumentations opencv-python

# Detectors
python3 -m pip install albumentations torch torchvision

# Classifiers + Detectors
python3 -m pip install albumentations opencv-python torch torchvision
```

Once these packages are all successfully installed, you'll be ready to run Python example
applications using your LEIP artifacts on your AWS instance!

_Please note that this dependencies list was up to date as of May 2024, but you can also
check the documentaion inside `$LEIP_DIRECTORY/example-applications` in case anything has
changed._

## Step Five: Downloading the LEIP Tooling

With our instance ready to go, we can now pull down our Docker containers ready
to start our LEIP environment. To do this we first need to login to the Latent AI
Docker registry:

```bash
docker login repository.latentai.com
```

You'll be prompted for a username and password. Please use your registry key/secret
to authenticate. If all goes well you should be greeted with `Login Succeeded`.

With access to the registry, we can now pull down the image we'll be using to run
LEIP. This can be done by using the Docker Compose file included in the tutorials
repository.

```bash
docker compose -f $LEIP_DIRECTORY/tutorials/environment/docker-compose.yml pull
```

This step will take a while, so grab a cup of coffee and come back in a few minutes!

Once everything has been pulled down successfully, congratulations! You now have an EC2
instance set up for remote Jupyter access, with all LEIP tools available as containers
locally.

To learn how to use these tools to run through our Getting Started notebook and begin
running inference using LEIP models, please see the [corresponding guide](./getting-started-with-the-leip-aws-ami.md).

