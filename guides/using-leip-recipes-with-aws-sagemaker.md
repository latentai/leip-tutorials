
## Using LEIP Recipes with Amazon Sagemaker

Amazon Web Services (AWS) Sagemaker provides a simple way to train a model on Amazon's hardware using Docker containers and images; all that is required is data input. This content is based on [LEIP Recipes](https://docs.latentai.io/leip-recipes/v2.5.0/).

This tutorial sets up AWS Sagemaker to train the LEIP Recipe `yolov5_L_RT` using the `soda10m` dataset. Adapting for other recipes and data are straightforward. Please refer to the Quantify POC benchmarking LEIP and Sagemaker NEO at [https://gitlab.com/latentai/collab/quantiphi](https://gitlab.com/latentai/collab/quantiphi).

### Set Up Your S3 Bucket

Create a new S3 bucket to hold the data, or an existing one can be used if you already have data in S3. Input the data in the S3 bucket, and tweak the structure to work with Sagemaker.

Download the `soda10m` dataset from the [website](https://soda-2d.github.io/download.html) and upload the data into Amazon S3. The directory tree for the new S3 bucket is as follows:

```
<bucket>
├── input
│   └── data              # from SSLAD-2D/labeled
│       ├── annotations
│       ├── train
│       └── val
└── output
```

The `input` and `output` directories can be placed in a subdirectory named `soda10m` in case you plan on adding more datasets in future. Both of these directories are used for mounts in Sagemaker, so it is important that you structure your data as closely to this as possible.

This directory structure is not critical for AWS Sagemaker because you can map individual directories into different paths in your containers. However it does make it easier to test the images as you can mount data directly from S3 without having to shift the directories.

### Set Up a Working Directory

Create a directory on your local machine once the data has been loaded on the S3 bucket. This is done to keep the Docker context clean. The following files will be created in the new directory for this tutorial:

```
<directory>
├── Dockfile                # the Dockerfile for your training image
├── license.txt             # text file containing your license key
├── soda10m_config.yaml     # the configuration for your training data
└── train                   # the training script required by AWS Sagemaker
```

Each of these files will be used by Docker at a later point, so everything should be placed in the same directory. For now, place your license key inside `license.txt` so it can be authorized from inside the image.

Create your data configuration inside `soda10m_config.yaml` to point to the locations you need on S3. Follow the instructions for [BYOD](https://docs.latentai.io/leip-recipes/v2.5.0/BYOD:-Training-With-Your-Own-Data.1330741338.html)) in the LEIP Recipe documentation.

The root path of your configured data should be listed as `/opt/ml/input/data`, as this is where Sagemaker will mount it. The exact directory paths are configured later in the Sagemaker UI, but the root will always be the same: `/opt/ml/input/data`.

### Construct Your Dockerfile

The next step is preparing the Dockerfile, the main requirement of Sagemaker. We will copy the example from the AWS Documentation but change the base image to be a LEIP image:

```dockerfile
FROM registry.latentai.io/leip-sdk/gpu-cuda10.2:2.5.0

# Add training dependencies and programs
COPY /license.txt /root/.LICENSE_KEY
COPY /soda10m_config.yaml /latentai/custom-configs/data/soda10m_config.yaml

# Add a script that SageMaker will run
# Set run permissions
# Prepend program directory to $PATH
COPY /train /opt/program/train
RUN chmod 755 /opt/program/train
ENV PATH=/opt/program:${PATH}
```

Copy the license key and the data configuration directly into the image. This simple Dockerfile gives you access to everything you need for your LEIP Recipe, and all of your local files are mounted into the container.

## Build Your Training Script

Your training script is the main focus of using AWS Sagemaker; it should use your sample data to build and train your model. The training script produces a trained model and mounts it in `/opt/ml/model` inside the container. This directory is automatically archived into S3 by Sagemaker. Data can be input into the directory, and it will be compressed and transferred into a labeled directory on S3 based on the name of any associated training job.

The following is a sample script and solid starting point:

```bash
#!/bin/bash
set -e
cd /latentai

# BYOD related steps
af --config-name=yolov5_L_RT \
    data=soda10m_config \
    'hydra.searchpath=\[file://custom-configs\]' \
    command=train \
    task.moniker="BYOD_recipe"

# Exporting your model steps
af --config-name=yolov5_L_RT \
    command=export

# Compile and optimization steps
export PYTHONPATH=/latentai/recipes/yolov5_L_RT/evaluation/utils:$PYTHONPATH
leip pipeline \
   --input_path $(ls artifacts/\*.pt) \
   --output_path ./workspace/recipes_output/yolov5_L_RT \
   --config_path ./recipes/yolov5_L_RT/pipeline_x86_64_cuda.json

# Archiving steps
cd recipes/yolov5_L_RT
sh ./create-artifacts-tar.sh
mv model-artifacts.tar.gz /opt/ml/model/model-artifacts.tar.gz

# Re-archiving for AWS steps
cd /opt/ml/model/
tar xvzf model-artifacts.tar.gz
rm model-artifacts.tar.gz
```

If you have any failures in your script, you can write the errors to `/opt/ml/output/failure` (which is a file). This will then be read by the Sagemaker UI to provide a failure reason without having to sift through logs looking for the issue.

### Upload Your Training Image

Once you have followed all the previous steps, you can build your image using the following command:

```
docker build -t my-training-image:dev .
```

If you want to test your image locally before you put it in Sagemaker, you can replicate what Sagemaker will do using the following command:

```bash
docker run \
  --rm \
  --gpus all \
  --volume "<path_to_input>:/opt/ml/input:ro" \
  --volume "<path_to_output>:/opt/ml/model" \
  --volume "<path_to_output>:/opt/ml/output" \
  my-training-image:dev \
  train
```

When everything is satisfactory, you will need to build and upload your training image directly to Amazon Elastic Container Registry (ECR). If you’re unfamiliar with doing this, the official documentation has a [reasonable guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html) on doing so.

### Running Your Training Job

When your image is uploaded, you can go into AWS Sagemaker and begin to set up a training job. This is done by navigating to `Training > Trainings Jobs` in the sidebar, then `Create Training Job` in the top right.

Complete the form that is provided. Ensure to provide your ECR path for your image and that you add channels for all your S3 data. If you refer to the S3 tree at the top of these notes, you would create a channel for each of `annotations`, `train` and `val`. These then get mounted into your container at the following paths:

*   `/opt/ml/input/data/annotations`
    
*   `/opt/ml/input/data/train`
    
*   `/opt/ml/input/data/val`
    

Direct your output to your output directory in S3. Your model data and any artifacts should end up in this directory, and any logs or failures will appear in either AWS CloudWatch or the Sagemaker UI (if you wrote a failure file). This also includes the model bindings you can use for inference should you wish to create a Sagemaker algorithm in the future. Logs for your training job can be found in AWS CloudWatch in the log group named `/aws/sagemaker/TrainingJobs`; check that log for progress and output from the image.

## Supporting AWS Sagemaker by Default

The idea here is that we include a pretty standard training script in the `leip-sdk` images and customize on the fly instead of each user having to create their own image. This would mean that we could ship an image which accepts the following at configuration time:

*   The license key of the user, so it is not baked into the image.
    
*   The name of the recipe being used for the model, instead of hardcoding.
    
*   The name of the custom data configuration, so you can use the same image with multiple different datasets with different structures.
    

Combining all of this allows us to include a `train` script in LEIP images so that Sagemaker training is supports out of the box features,  and a user can just take a `leip-sdk` image directly without having to modify it themselves.

There are a couple of ways to make this happen. The first is super simple and pretty much already supported (as far as I can tell), which is providing these values through Environment Variables. The catch is that the Sagemaker UI doesn’t expose adding Environment Variables for training jobs; only the CLI, so it’s not super user friendly.

The second is pretty much the same gist; you can provide Hyperparameter values in the Sagemaker UI which get mounted at `/opt/ml/input/config/hyperparameters.json` inside the container. We could probably hijack this if we wanted, but it feels a bit awkward to do it this way.

The final option is to have the user mount a configuration file, which our `train` script reads and uses to customize exactly what's going on. This is of course the most involved, but it’s probably the most flexible for the future. All of my prototypes were done in Bash and a really basic configuration syntax, but we could easily write the `train` script in something like Python and a YAML configuration as we already have a Python environment available to us in those images.

The idea is pretty simple and I have a working prototype for it; you make a couple of slight tweaks to your S3 structure (from the one shown earlier) to make it looks like this:

```
<bucket>
├── input
│   └── data
│       ├── annotations                     # from SSLAD-2D/labeled
│       ├── config
│       │   ├── config.cfg
│       │   └── data
│       │       └── soda10m_config.yaml
│       ├── training                        # from SSLAD-2D/labeled
│       └── val                             # from SSLAD-2D/labeled
└── output
```

Inside the configuration directory you place a new configuration file alongside the data configuration file which was previously included in the Docker image directly. As mentioned, this configuration file could be in whatever format we wanted but because I used Bash, as I apparently hate myself, I used a configuration with a basic format:

```properties
license_key=<key>
recipe=yolov5_L_RT
data_config=soda10m_config
pipeline_config=pipeline_aarch64_cuda
```

There’s other stuff we can include in future if we want; one more compelling option might be logging configurations to integrate with tools like Neptune more easily. Another would be providing config for basics like environment variables a little more easily, etc.

An example training script for this setup would look something like the following (well, maybe a bit prettier). It reads the options out of the mounted `config.cfg` and trains a model based on them rather than including things in the image itself.

```bash
#!/bin/bash
set -e
cd /latentai

# Configuration reading
_config_get () {
    path="/opt/ml/input/data/config/config.cfg"
    value=$((grep -E "^${1}=" -m 1 "$path" 2> /dev/null || echo "VAR=__UNDEFINED__") | head -n 1 | cut -d '=' -f 2-)

    if \[ "${value}" != "__UNDEFINED__" \]; then
        printf -- "%s" "${value}";
    fi
}

# Configuration steps
RECIPE=$(_config_get "recipe")
DATA_CONFIG=$(_config_get "data_config")
PIPELINE_CONFIG=$(_config_get "pipeline_config")
export LICENSE_KEY=$(_config_get "license_key")

# Verification steps
if \[\[ -z "${RECIPE}" || -z "${PIPELINE_CONFIG}" \]\]; then
    echo '$RECIPE and $PIPELINE_CONFIG are required for training models' > /opt/ml/output/failure
    exit 1
fi

# BYOD related steps
if \[\[ -n "${DATA_CONFIG}" \]\]; then
  af --config-name=$RECIPE \
    data=$DATA_CONFIG \
    'hydra.searchpath=\[file:///opt/ml/input/data/config\]' \
    command=train \
    task.moniker="BYOD_recipe"
fi

# Exporting your model steps
af --config-name=$RECIPE \
    command=export

# Compile and optimization steps
export PYTHONPATH=/latentai/recipes/$RECIPE/evaluation/utils:$PYTHONPATH
leip pipeline \
   --input_path $(ls artifacts/\*.pt) \
   --output_path ./workspace/recipes_output/$RECIPE \
   --config_path ./recipes/$RECIPE/$PIPELINE_CONFIG.json

# gather dependencies
cd recipes/$RECIPE
sh ./create-artifacts-tar.sh
mv model-artifacts.tar.gz /opt/ml/model/model-artifacts.tar.gz

# let AWS package for us
cd /opt/ml/model/
tar xvzf model-artifacts.tar.gz
rm model-artifacts.tar.gz
```

A lot of it is similar to the earlier script, it just adds configuration bindings for the naming and paths of certain values. Inclusion of a training script in the base image simplifies a lot of the user workflow. If this were in place, the workflow of a customer would look something similar to:

*   Upload their training data into AWS S3
    
*   Create whatever small configuration we need and drop it in S3
    
*   Configure the Latent AI image repository on AWS or mirror the LEIP image into their AWS ECR
    
*   Create a training job and mount it the config at (e.g) `/opt/ml/input/data/config`
    

If their data happens to already be available in S3, this would take only a couple of minutes to get a training job running for a LEIP Recipe. Compared to all of the tweaks with images earlier in this page, this basically only requires interacting with the AWS Sagemaker UI and filling in forms etc.

## Other Information

If there’s anything else I come across that might be useful, I’ll drop it in here so we don’t forget about them in case they’re of some benefit in future:

*   [https://sagemaker-workshop.com/custom/containers.html](https://sagemaker-workshop.com/custom/containers.html)
    
*   [https://github.com/aws/amazon-sagemaker-examples](https://github.com/aws/amazon-sagemaker-examples)
    
*   [https://docs.aws.amazon.com/sagemaker/latest/dg/your-algorithms-containers-inference-private.html](https://docs.aws.amazon.com/sagemaker/latest/dg/your-algorithms-containers-inference-private.html)
    

And because it has been mentioned a few times about instances, here are the instance families you can create training jobs with (I just pulled a list from the UI):

| **Instance Type** | **Minimum Sizing** | **Maximum Sizing** |
|-------------------|--------------------|--------------------|
| ml.c4             | xlarge             | 8xlarge            |
| ml.c5             | xlarge             | 18xlarge           |
| ml.c5n            | xlarge             | 18xlarge           |
| ml.g4dn           | xlarge             | 16xlarge           |
| ml.g5             | xlarge             | 48xlarge           |
| ml.m4             | xlarge             | 16xlarge           |
| ml.m5             | large              | 24xlarge           |
| ml.p2             | xlarge             | 16xlarge           |
| ml.p3             | 2xlarge            | 16xlarge           |
| ml.p3dn           | 24xlarge           | 24xlarge           |
| ml.p4d            | 24xlarge           | 24xlarge           |