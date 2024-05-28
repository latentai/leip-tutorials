# Getting Started with the LEIP AWS AMI

This guide will walk you through running the LEIP Getting Started notebook on a
preconfigured LEIP AWS AMI instance. We will also cover executing your optimized
LEIP models for inference via the LEIP Example Applications.

At the end of this tutorial, you should have a fairly good idea on how to work
with the various LEIP APIs as well as a good starting point for running inference
with LEIP artifacts.

## Requirements

This guide is suited for customers who have either:

* Been provided with an AWS instance running the LEIP AWS AMI
* Created an AWS instance following the LEIP AWS EC2 walkthrough

For anyone who does not fit the criteria above, please see the appropriate guide
to create your own instance before returning to this page.

This guide should generally take you around 30 minutes.

## Step One: Starting LEIP

If you have not already done so, please login to your AWS EC2 instance. If you had
an AWS instance provided to you, you should have also received both a private key
alongside the AWS IP. Please reach out to us if you are missing either of these!

```bash
ssh -i <path/to/your/key> ubuntu@<your-ip>
```

Once logged in, we can start the LEIP tooling. This is done by using the Docker
Compose definitions provided inside the LEIP tutorials repository:

```bash
docker compose -f $LEIP_DIRECTORY/tutorials/environment/docker-compose.yml up --pull never
```

This command will start the LEIP tooling in the foreground of your shell session,
which will allow us to verify that there are no errors in our configuration. You
can terminate this at any time by using `CTRL+C`.

You should be able to see messages from all containers in your terminal. The two
specific messages that you are looking for are:

```
leip-cf  | INFO:     Application startup complete.
leip-af  | [I 2024-04-12 18:28:42.804 ServerApp] Jupyter Server 2.14.0 is running at:
```

Please note that these will likely not appear next to each other as shown above, so
you may have to do a little searching. When you see these message, everything should
be running correctly. To verify this, you can visit the following in your browser:

* `http://<your-ip>:8888`
    * This is the main Jupyter server allowing you to run notebooks
* `http://<your-ip>:8080/api`
    * This checks the LEIP Compiler Framework endpoint is up and running
    * This will only be reachable if you configured port `8080` in your security group

If all looks good, you can continue to the next step! You may also wish to open up
another shell session to continue working while your tools are running, or you can
choose to run your containers in the background by passing the `-d` flag to the
`docker compose` command above.

## Step Two: Running the Getting Started Notebook

Most of this section will take place inside the Jupyter UI inside your browser, so
please navigate to `http://<your-ip>:8888`.

You'll be greeted with a welcome screen, which prompts you for a password or token
to access the main Jupyter server. If you are unsure which token has been configured
for your specific instance, you can run this inside your shell session:

```bash
echo $JUPYTER_TOKEN
```

After logging in with your token, you should be able to see a directory listing with
the `notebooks` directory. Go ahead and double click to open this directory. This
will show you several notebook files provided as tutorials for the LEIP platform.
For this guide we're going to double click the `GettingStarted.ipynb` file to open
up the notebook in a new tab.

At this point we're going to work through the notebook. It's recommended to take your
time here to read through the documentation in the notebook (at your own pace) to gain
a good understand of the LEIP tooling flow and how the various APIs work.

For the sake of this guide though, we're going to run the full notebook automatically.
You can do this by clicking `Run` in the toolbar and selecting `Run All Cells`. Each
block of Python code will execute and print the output inside the notebook UI so you
can follow along with the execution flow.

## Step Three: Deploying our LEIP Artifacts

_At this point you'll want to either open up a new shell or move the LEIP tooling to
the background as described in the end of Step One._

After our notebook has completed (which will likely take around 15 to 30 minutes),
we'll be left with several files for deployment with which we can run inference:

* Compiled model and metadata
    * `$LEIP_WORKSPACE/deploy/optimized_model.tar.gz`
* Test images
    * `$LEIP_WORKSPACE/deploy/road314.png`
* Application artifacts
    * `$LEIP_WORKSPACE/deploy/roadsign.txt`

If you were to use these for inference on another machine, you'd be able to archive
this directory and transfer it over. For the purposes of this guide we'll be running
inference on the same machine, so we just need to extract our model assets inside the
deployment directory:

```bash
sudo chown -R ubuntu:ubuntu $LEIP_WORKSPACE/deploy
cd $LEIP_WORKSPACE/deploy
tar xzvf optimized_model.tar.gz
```

This will give us access to our executable artifacts as well as all medatata which
may be relevant during inference. With all this done, we're ready to run inference!

## Step Four: Running a Python Inference

Now that our artifacts deployed and ready to go, we can now use the Python example from
the LEIP Example Applications to run a basic inference using our deployment directory.

To do this we can execute the Python example application using our deployed files to
run inference on our test image. There are several applications to choose from, but
for now we'll run a sample image detection:

```bash
cd $LEIP_DIRECTORY/example-applications/detectors/python_inference

python3 infer.py \
	--model_binary_path $LEIP_WORKSPACE/deploy \
	--input_image_path $LEIP_WORKSPACE/deploy/road314.png \
	--labels $LEIP_WORKSPACE/deploy/roadsign.txt
```

Assuming everything goes smoothly, after a short period of time we'll be given a
summary of our inference results. This provides us with various statistics about
our inference, as well as the path of our annotated test image:

```json
{
  "UUID": "c2b13e65-4d5e-4db2-bc0e-86ddbf561891",
  "Precision": "float32",
  "Device": "DLDeviceType::kDLCUDA",
  "Input Image Size": [
    400,
    300,
    3
  ],
  "Model Input Shapes": [
    [
      1,
      3,
      512,
      512
    ]
  ],
  "Model Input Layouts": [
    "NCHW"
  ],
  "Average Preprocessing Time ms": {
    "Mean": 4.478,
    "std_dev": 0.713
  },
  "Average Inference Time ms": {
    "Mean": 2.52,
    "std_dev": 0.069
  },
  "Average Total Postprocessing Time ms": {
    "Mean": 1.087,
    "std_dev": 0.137
  },
  "Total Time ms": 8.085,
  "Annotated Image": "$LEIP_WORKSPACE/deploy/road314-2024-05-19 04:49:07.257987.png"
}
```

With our inference complete, congratulations! You have now successfully compiled
and optimized a model using LEIP, and then used this model to run image detection
and produced an annotated output!

_If you'd like to compare the input and output images, you can navigate into the
`deploy` directory inside the Jupyter UI. This will allow you to compare your image
before and after annotation._
