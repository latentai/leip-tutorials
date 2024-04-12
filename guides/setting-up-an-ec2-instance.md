# Running LEIP on an AWS EC2 Instance

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

This guide should take you no longer than around 30 minutes.

## Step One: Creating your Amazon Instance

We'll begin by logging into AWS EC2 and creating a new instance. From the AWS
EC2 console, select "Launch Instances" from the top right of the screen. This
will drop you into a form to create your instance. You can use the following
as a guide for filling out this form:

1. You can name the instance whatever you want
2. Under "Application and OS Images" we recommend to use a Quick Start AMI
    a. Select "Ubuntu" in the Quick Start menu
    b. Expand the dropdown and search for "Deep Learning Base OSS Nvidia"
    c. There should be a single result, so select this as your base image
    d. Please see the [release notes](https://aws.amazon.com/releasenotes/aws-deep-learning-base-gpu-ami-ubuntu-20-04/)
       for the list of supported instance types
3. Under "Instance Type" you can select your instance type and size
    a. If you don't have specific requirements, you should select the
       `g4dn.xlarge` instance type.
    b. If picking your own instance type, please make sure it's supported
       by checking the notes in `2d`.
4. Under "Key Pair" you can select/create a key pair for your instance
    a. Although you can skip this step, it is highly recommended to secure
       access to your instance by assigning a key pair.
    b. This may not be a concern if you only allow SSH from specific IPs.
5. Under "Network Settings", make sure to click "Edit" to open the advanced
   configuration settings.
    a. Make sure that "Auto-assign public IP" is set to "Enable"
    b. Create a new security group named "leip-sg"
    c. There are several recommended rules to add to this group:
        i. Allow `SSH` on port `22` from either your IP or anywhere (if using a
           key pair).
        ii. Allow `Custom TCP` on port `8888` if you wish to access Jupyter and
            use Jupyter notebooks directly from your workstation
        iii. Allow `Custom TCP` on port `8080` if you wish to access LEIP CF via
             the LEIP Server from your workstation
6. Under "Configure Storage" choose your volume size
    a. We recommend starting with at least `256gb` on a `gp3` volume
    b. If you anticipate you'll need more, feel free to raise this higher now to
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

```
ssh -i <path/to/your/key> ubuntu@<your-ip>
```

Once you're inside the system, you can continue on to the next step. Please note
that it might take a couple of minutes for the instance to fully start and your
login to work successfully.

## Step Two: Configuring your Environment

Once you're logged into your machine, the first step is to make sure everything
is up to date. This is important as it will update Nvidia drivers, as well as the
Nvidia Docker toolkit, to the the latest versions.

```
sudo apt-get update
sudo apt-get upgrade
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
able to access your Jupyter installation. Please note that you will need to either
`source ~/.bashrc` or log out/in to refresh your current shell variables.

The last step we need to take is cloning the LEIP tutorials repository onto the
machine. Even if you're not planning on using the notebooks inside this repository,
we'll use it to start our Docker containers:

```bash
sudo mkdir -p $LEIP_WORKSPACE
sudo git clone https://github.com/latentai/leip-tutorials $LEIP_DIRECTORY/tutorials
sudo chown -R ubuntu:ubuntu $LEIP_DIRECTORY
```

Please note that you can set up your workspace and tutorials in whatever location
you wish, but the above are the recommended locations to keep everything organized.

## Step Three: Starting LEIP

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
Once you have your images pulled down, you can start your LEIP components:

```bash
docker compose -f $LEIP_DIRECTORY/tutorials/environment/docker-compose.yml up
```

For the first run it is recommended to run without the `-d` flag, which runs the
containers in the background. Running in the foreground for now allows us to track
any errors in our configuration. You can `CTRL+C` to stop it at any point.

You should see logs from all containers in your console. You should be able to see
the following messages from each of the containers:

```
leip-cf  | INFO:     Application startup complete.
leip-af  | [I 2024-04-12 18:28:42.804 ServerApp] Jupyter Server 2.14.0 is running at:
```

If you see warnings about your license key, you need to revisit Step Two and ensure
that you can see your variables in your current shell session. To verify that every
has started correctly, you can visit the following in your browser:

* `http://<your-ip>:8888`
    * This is the main Jupyter server allowing you to run notebooks
* `http://<your-ip>:8080/api`
    * This checks the LEIP Compiler Framework endpoint is up and running
    * This will only be reachable if you configured port `8080` in your security group

If all looks good, you can continue to the next step!

## Step Four: Finishing Up

If you've successfully reached this step, you're practically done!

Assuming everything is working as expected from Step Three, you can now `CTRL+C` to
stop your current running containers, and then re-run the previous command but this
time adding the `-d` flag at the end. This will run your containers in the background
so they will keep running even if you log out from the instance.

That's it! You now have an EC2 instance set up for remote Jupyter access, with all
LEIP tools running as containers in the background. If you wish to, you can now run
through our Getting Started notebook in the Jupyter UI!


