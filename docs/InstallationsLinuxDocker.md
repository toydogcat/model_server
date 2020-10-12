# Installing OpenVINO&trade; Model Server for Linux using Docker Container

## Introduction

 OpenVINO&trade; Model Server is a serving system for machine learning models.  OpenVINO&trade; Model Server  makes it easy to deploy new algorithms and experiments, while keeping the same server architecture and APIs. This guide will help you install  OpenVINO&trade; Model Server for Linux through docker containers.

## System Requirements

### Hardware 
* 6th to 10th generation Intel® Core™ processors and Intel® Xeon® processors.
* Intel® Neural Compute Stick 2.
* Intel® Vision Accelerator Design with Intel® Movidius™ VPUs.

### Operating Systems

* Ubuntu 18.04.x long-term support (LTS), 64-bit.

### Overview 

This guide provides step-by-step instructions on how to install OpenVINO&trade; Model Server for Linux using Docker Container including a Quick Start guide. Links are provided for different compatible hardwares. Following instructions are covered in this :

- <a href="#ExistingDocker">Installing OpenVINO&trade; Model Server with existing Docker Container</a>
- <a href="#quickstart">Quick Start Guide for OpenVINO&trade; Model Server</a>
- <a href="#singlemodel">Starting Docker Container with a Single Model
- <a href="#configfile">Starting Docker container with a configuration file for multiple models</a>
- <a href="#sourcecode">Building the OpenVINO&trade; Model Server Image </a>
- <a href="#ncs">Starting Docker container with Neural Compute Stick</a>
- <a href="#hddl">Starting Docker container with HDDL Plugin </a>
- <a href="#gpu">Starting Docker container with GPU</a>
- <a href="#multiplugin">Starting Docker Container using Multi-Device Plugin</a>
- <a href="#heteroplugin">Starting Docker Container using Heterogenous Plugin </a>



##  Installing OpenVINO&trade; Model Server with existing Docker Container<a name="ExistingDocker"></a>

A quick start guide to install model server and run it with face detection model is provided below. It includes scripts to query the gRPC endpoints and save results.

For additional endpoints, refer the [REST API](./ModelServerRESTAPI.md).

### Quick Start Guide <a name="quickstart"></a>

```bash
 # Pull the latest version of OpenVINO&trade; Model Server from Dockerhub - 
docker pull openvino/model_server:latest

#  Download model files into a separate directory
# OpenVINO&trade; Model Server requires models in model repositories. Refer to this link (Preparation of Models) or run following command to get started with an example

curl --create-dirs https://download.01.org/opencv/2020/openvinotoolkit/2020.4/open_model_zoo/models_bin/3/face-detection-retail-0004/FP32/face-detection-retail-0004.xml https://download.01.org/opencv/2020/openvinotoolkit/2020.4/open_model_zoo/models_bin/3/face-detection-retail-0004/FP32/face-detection-retail-0004.bin -o model/face-detection-retail-0004.xml -o model/face-detection-retail-0004.bin

#  Start the container serving gRPC on port 9000 
docker run -d -v <folder_with_downloaded_model>:/models/face-detection/1 -p 9000:9000 openvino/model_server:latest \
--model_path /models/face-detection --model_name face-detection --port 9000 --log_level DEBUG --shape auto

#  Download the example client script to test and run the gRPC 
curl https://raw.githubusercontent.com/openvinotoolkit/model_server/main/example_client/client_utils.py -o client_utils.py https://raw.githubusercontent.com/openvinotoolkit/model_server/main/example_client/face_detection.py -o face_detection.py  https://raw.githubusercontent.com/openvinotoolkit/model_server/main/example_client/client_requirements.txt -o client_requirements.txt

# Download an image to be analyzed
curl --create-dirs https://raw.githubusercontent.com/openvinotoolkit/model_server/main/example_client/images/people/people1.jpeg -o images/people1.jpeg

# Install python client dependencies
pip install -r client_requirements.txt

#  Create a directory for results
mkdir results

# Run inference and store results in the newly created folder
python face_detection.py --batch_size 1 --width 600 --height 400 --input_images_dir images --output_dir results

# Check results folder for image with inference drawn over it.
```


## Detailed steps to install OpenVINO&trade; Model Server using Docker container

### Install Docker

Install Docker for Ubuntu 18.04 using the following link :

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### Pulling OpenVINO&trade; Model Server Image

After Docker installation you can pull the OpenVINO&trade; Model Server image. Open Terminal and run following command :

```bash
docker pull openvino/model_server:latest

```
**You can also build your own image using steps in - <a href="#sourcecode">Build OpenVINO&trade; Model Server Image </a>.**

### Running the OpenVINO&trade; Model Server image for **Single** Model <a name="singlemodel"></a>

Follow the [Preparation of Model guide](./PreparingModelsRepository.md) before running the docker image.

Run the OpenVINO&trade; Model Server by running the following command: 

```
docker run -d -v <folder_with_downloaded_model>:/models/face-detection/1 -e LOG_LEVEL=DEBUG -p 9000:9000 -p 9001:9001 openvino/model_server:latest \
--model_path path_to_model --model_name model_name --port 9000 --rest_port 9001 --shape auto
```

#### Configuration Arguments for running the OpenVINO&trade; Model Server :

- --rm - Remove the container when exiting the Docker container.
- -d - Run the container in the background.
- -v - Defines how to mount the models folder in the Docker container.
- -p - Exposes the model serving port outside the Docker container.
- openvino/model_server:latest - Represents the image name. This varies by tag and build process. The ovms binary is the Docker entry point. See the full list of ovms tags.
- --model_path - Model location. This can be a Docker container that is mounted during start-up or a Google* Cloud Storage path in format gs://<bucket>/<model_path> or AWS S3 path s3://<bucket>/<model_path>. See the requirements below for using a cloud storage.
- --model_name - The name of the model in the model_path.
- --port - gRPC server port.
- --rest_port - REST server port.


*Notes*

- Publish the container's port to your host's **open ports**.
- In above command port 9000 is exposed for gRPC and port 9001 is exposed for REST API calls.
- For preparing and saving models to serve with OpenVINO&trade; Model Server refer [this](./PreparingModelsRepository.md).
- Add model_name for the client gRPC/REST API calls.





### Configuration Parameters :

<details><summary>Model configuration options</summary>

| Option  | Value format  | Description  | Required |
|---|---|---|---|
| `"model_name"/"name"` | `string` | model name exposed over gRPC and REST API.(use `model_name` in command line, `name` in json config)   | &check;|
| `"model_path"/"base_path"` | `"/opt/ml/models/model"`<br>"gs://bucket/models/model"<br>"s3://bucket/models/model"<br>"azure://bucket/models/model" | If using a Google Cloud Storage, Azure Storage or S3 path, see the requirements below.(use `model_path` in command line, `base_path` in json config)  | &check;|
| `"shape"` | `tuple, json or "auto"` | `shape` is optional and takes precedence over `batch_size`. The `shape` argument changes the model that is enabled in the model server to fit the parameters. <br><br>`shape` accepts three forms of the values:<br>* `auto` - The model server reloads the model with the shape that matches the input data matrix.<br>* a tuple, such as `(1,3,224,224)` - The tuple defines the shape to use for all incoming requests for models with a single input.<br>* A dictionary of tuples, such as `{input1:(1,3,224,224),input2:(1,3,50,50)}` - This option defines the shape of every included input in the model.<br><br>Some models don't support the reshape operation.<br><br>If the model can't be reshaped, it remains in the original parameters and all requests with incompatible input format result in an error. See the logs for more information about specific errors.<br><br>Learn more about supported model graph layers including all limitations at [docs_IE_DG_ShapeInference](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_ShapeInference.html). ||
| `"batch_size"` | `integer / "auto"` | Optional. By default, the batch size is derived from the model, defined through the OpenVINO Model Optimizer. `batch_size` is useful for sequential inference requests of the same batch size.<br><br>Some models, such as object detection, don't work correctly with the `batch_size` parameter. With these models, the output's first dimension doesn't represent the batch size. You can set the batch size for these models by using network reshaping and setting the `shape` parameter appropriately.<br><br>The default option of using the Model Optimizer to determine the batch size uses the size of the first dimension in the first input for the size. For example, if the input shape is `(1, 3, 225, 225)`, the batch size is set to `1`. If you set `batch_size` to a numerical value, the model batch size is changed when the service starts.<br><br>`batch_size` also accepts a value of `auto`. If you use `auto`, then the served model batch size is set according to the incoming data at run time. The model is reloaded each time the input data changes the batch size. You might see a delayed response upon the first request.<br>  ||
| `"model_version_policy"` | <code>{"all": {}}<br>{"latest": { "num_versions": Integer}<br>{"specific": { "versions":[1, 3] }}</code> | Optional.<br><br>The model version policy lets you decide which versions of a model that the OpenVINO Model Server is to serve. By default, the server serves the latest version. One reason to use this argument is to control the server memory consumption.<br><br>The accepted format is in json.<br><br>Examples:<br><code>{"latest": { "num_versions":2 } # server will serve only ywo latest versions of model<br><br>{"specific": { "versions":[1, 3] }} # server will serve only 1 and 3 versions of given model<br><br>{"all": {}} # server will serve all available versions of given model ||
| `"plugin_config"` | json with plugin config mappings like`{"CPU_THROUGHPUT_STREAMS": "CPU_THROUGHPUT_AUTO"}` |  List of device plugin parameters. For full list refer to [OpenVINO documentation](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_Supported_Devices.html) and [performance tuning guide](./PerformanceInformation.md)  ||
| `"nireq"`  | `integer` | The size of internal request queue. When set to 0 or no value is set value is calculated automatically based on available resources.||
| `"target_device"` | `"CPU"/"HDDL"/"GPU"/"NCS"/"MULTI"/"HETERO"` |  Device name to be used to execute inference operations. Refer to AI accelerators support below. ||

#### To know more about batch size and shape parameters refer [Batch Size and Shape document](./ShapeAndBatchSize.md)

</details>


<details><summary>Server configuration options</summary>

Configuration options for server are defined only via command line options and determine configuration common for all served models. 

| Option  | Value format  | Description  | Required |
|---|---|---|---|
| `port` | `integer` | Number of the port used by gRPC sever. | &check;|
| `rest_port` | `integer` |  Number of the port used by HTTP server (if not provided or set to 0, HTTP server will not be launched). ||
| `grpc_workers` | `integer` |  Number of the gRPC server instances (should be from 1 to CPU core count). Default value is 1 and it's optimal for most use cases. Consider setting higher value while expecting heavy load. ||
| `rest_workers` | `integer` |  Number of HTTP server threads. Effective when `rest_port` > 0. Default value is 24. ||
| `file_system_poll_wait_seconds` | `integer` |  Time interval between config and model versions changes detection in seconds. Default value is 1. Zero value disables changes monitoring. ||
| `log_level` | `"DEBUG"/"INFO"/"ERROR"` |  Serving logging level ||
| `log_path` | `string` |  Optional path to the log file. ||


</details>

<details><summary>Cloud Storage Requirements </summary>

### Azure Cloud Storage path requirements

Add the Azure Storage path as the model_path and pass the Azure Storage credentials to the Docker container.

To start a Docker container with support for Azure Storage paths to your model use the AZURE_STORAGE_CONNECTION_STRING variable. This variable contains the connection string to the AS authentication storage account.

Example connection string is: AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=azure_account_name;AccountKey=smp/hashkey==;EndpointSuffix=core.windows.net"

Example command with blob storage az://<bucket>/<model_path> :
```
docker run --rm -d  -p 9001:9001 \
-e AZURE_STORAGE_CONNECTION_STRING=“${AZURE_STORAGE_CONNECTION_STRING}” \
openvino/model_server:latest \
--model_path az://bucket/model_path --model_name as_model --port 9001
```

Example command with file storage azfs://<share>/<model_path> :

```
docker run --rm -d  -p 9001:9001 \
-e AZURE_STORAGE_CONNECTION_STRING=“${AZURE_STORAGE_CONNECTION_STRING}” \
openvino/model_server:latest \
--model_path azfs://share/model_path --model_name as_model --port 9001
Add -e "http_proxy=$http_proxy" -e "https_proxy=$https_proxy" to docker run command for proxy cloud storage connection.
```

By default the https_proxy setting will be used. If you want to use http_proxy please set the AZURE_STORAGE_USE_HTTP_PROXY environment variable to any value and pass it to the container.

###  Google Cloud Storage path requirements

Add the Google Cloud Storage path as the model_path and pass the Google Cloud Storage credentials to the Docker container.
Exception: This is not required if you use GKE kubernetes cluster. GKE kubernetes clusters handle authorization.

To start a Docker container with support for Google Cloud Storage paths to your model use the GOOGLE_APPLICATION_CREDENTIALS variable. This variable contains the path to the GCP authentication key.

Example command with gs://<bucket>/<model_path>:
```
docker run --rm -d  -p 9001:9001 \
-e GOOGLE_APPLICATION_CREDENTIALS=“${GOOGLE_APPLICATION_CREDENTIALS}” \
-v ${GOOGLE_APPLICATION_CREDENTIALS}:${GOOGLE_APPLICATION_CREDENTIALS} \
openvino/model_server:latest \
--model_path gs://bucket/model_path --model_name gs_model --port 9001
AWS S3 and Minio storage path requirements
```

Add the S3 path as the model_path and pass the credentials as environment variables to the Docker container.

Example command with s3://<bucket>/<model_path> :

```
docker run --rm -d -p 9001:9001 \
-e AWS_ACCESS_KEY_ID=“${AWS_ACCESS_KEY_ID}” \
-e AWS_SECRET_ACCESS_KEY=“${AWS_SECRET_ACCESS_KEY}” \
-e AWS_REGION=“${AWS_REGION}” \
-e S3_ENDPOINT=“${S3_ENDPOINT}” \
openvino/model_server:latest \
--model_path s3://bucket/model_path --model_name s3_model --port 9001

```
</details>

## Starting docker container with a configuration file for **multiple** models <a name="configfile"></a>

To use a container that has several models, you must use a model server configuration file that defines each model. The configuration file is in JSON format.
In the configuration file, provide an array, model_config_list, that includes a collection of config objects for each served model. For each config object include, at a minimum, values for the model name and the base_path attributes.

Example configuration file :
```json
{
   "model_config_list":[
      {
         "config":{
            "name":"model_name1",
            "base_path":"/opt/ml/models/model1",
            "batch_size": "16"
         }
      },
      {
         "config":{
            "name":"model_name2",
            "base_path":"/opt/ml/models/model2",
            "batch_size": "auto",
            "model_version_policy": {"all": {}}
         }
      },
      {
         "config":{
            "name":"model_name3",
            "base_path":"gs://bucket/models/model3",
            "model_version_policy": {"specific": { "versions":[1, 3] }},
            "shape": "auto"
         }
      },
      {
         "config":{
             "name":"model_name4",
             "base_path":"s3://bucket/models/model4",
             "shape": {
                "input1": "(1,3,200,200)",
                "input2": "(1,3,50,50)"
             },
             "plugin_config": {"CPU_THROUGHPUT_STREAMS": "CPU_THROUGHPUT_AUTO"}
         }
      },
      {
         "config":{
             "name":"model_name5",
             "base_path":"s3://bucket/models/model5",
             "shape": "auto",
             "nireq": 32,
             "target_device": "HDDL",
         }
      }
   ]
}
```

When the config file is present, the docker container can be started in a similar manner as a single model. Keep in mind that models with cloud storage path require specific environmental variables set. Refer to cloud storage requirements below :

```bash

docker run --rm -d  -v /models/:/opt/ml:ro -p 9001:9001 -p 8001:8001  -v <config.json>:/opt/ml/config.json ovms:latest \
--config_path /opt/ml/config.json --port 9001 --rest_port 8001

```

*Note* :  Follow the below model repository structure for multiple models:

```bash
models/
├── model1
│   ├── 1
│   │   ├── ir_model.bin
│   │   └── ir_model.xml
│   └── 2
│       ├── ir_model.bin
│       └── ir_model.xml
└── model2
    └── 1
        ├── ir_model.bin
        ├── ir_model.xml
        └── mapping_config.json
```

here the numerical values depict the version number of the model.

To know more about OpenVINO&trade; Model Server , you can refer [Version Policy](./ModelVersionPolicy.md) document.


##  Building the OpenVINO&trade; Model Server Image<a name="sourcecode"></a>

To build your own image, use the following command in the [git repository root folder](https://github.com/openvinotoolkit/model_server), replacing DLDT_PACKAGE_URL=<URL> with the URL to OpenVINO Toolkit package that you can get after registration on OpenVINO™ Toolkit website.

```bash
   make docker_build DLDT_PACKAGE_URL=<URL>
```
called from the root directory of the repository.

It will generate the images, tagged as :

- openvino/model_server:latest - with CPU, NCS and HDDL support
- openvino/model_server-gpu:latest - with CPU, NCS, HDDL and iGPU support
as well as a release package (.tar.gz, with ovms binary and necessary libraries), in a ./dist directory.

*The release package is compatible with linux machines on which glibc version is greater than or equal to the build image version. For debugging, an image with a suffix -build is also generated (i.e. openvino/model_server-build:latest).*

Note: Images include OpenVINO 2021.1 release.

## Running OpenVINO&trade; Model Server with AI Accelerators

###  Starting docker container with Neural Compute Stick<a name="ncs"></a>

Plugin for [Intel® Movidius™ Neural Compute Stick](https://software.intel.com/en-us/neural-compute-stick), starting from 
version 2019 R1.1 is distributed both in a binary package and [source code](https://github.com/opencv/dldt). 
You can build the docker image of OpenVINO Model Server, including Myriad plugin, using any form of the OpenVINO toolkit distribution:
- `make docker_build_bin dldt_package_url=<url>` 
- `make docker_build_apt_ubuntu`

Neural Compute Stick must be visible and accessible on host machine. You may need to update udev 
rules:
<details>
<summary><i>Updating udev rules</i></summary>
</br>

1. Create file __97-usbboot.rules__ and fill it with:

```
   SUBSYSTEM=="usb", ATTRS{idProduct}=="2150", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1" 
   SUBSYSTEM=="usb", ATTRS{idProduct}=="2485", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
   SUBSYSTEM=="usb", ATTRS{idProduct}=="f63b", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
```   
2. In the same directory execute following: 
 ```
   sudo cp 97-usbboot.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   sudo ldconfig
   rm 97-usbboot.rules
```

</details>
</br>

3. To start server with NCS you can use command similar to:

```
docker run --rm -it --net=host -u root --privileged -v /opt/model:/opt/model -v /dev:/dev -p 9001:9001 \
ovms:latest --model_path /opt/model --model_name my_model --port 9001 --target_device MYRIAD
```

`--net=host` and `--privileged` parameters are required for USB connection to work properly. 

`-v /dev:/dev` mounts USB drives.

A single stick can handle one model at a time. If there are multiple sticks plugged in, OpenVINO Toolkit 
chooses to which one the model is loaded. 

### Starting docker container with HDDL<a name="hddl"></a>

Plugin for High-Density Deep Learning (HDDL) accelerators based on [Intel Movidius Myriad VPUs](https://www.intel.ai/intel-movidius-myriad-vpus/#gs.xrw7cj)
is distributed only in a binary package. You can build the docker image of OpenVINO Model Server, including HDDL plugin, using OpenVINO toolkit binary distribution :
- `make docker_build_bin dldt_package_url=<url>` 

In order to run container that is using HDDL accelerator, _hddldaemon_ must
 run on host machine. It's  required to set up environment 
 (the OpenVINO package must be pre-installed) and start _hddldaemon_ on the
  host before starting a container. Refer to the steps from [OpenVINO documentation](https://docs.openvinotoolkit.org/latest/_docs_install_guides_installing_openvino_docker_linux.html#build_docker_image_for_intel_vision_accelerator_design_with_intel_movidius_vpus).

To start server with HDDL you can use command similar to ss:

```
docker run --rm -it --device=/dev/ion:/dev/ion -v /var/tmp:/var/tmp -v /opt/model:/opt/model -p 9001:9001 \
ovms:latest --model_path /opt/model --model_name my_model --port 9001 --target_device HDDL --nireq 16
```

`--device=/dev/ion:/dev/ion` mounts the accelerator.

`-v /var/tmp:/var/tmp` enables communication with _hddldaemon_ running on the
 host machine

### Starting docker container with GPU<a name="gpu"></a>

The GPU plugin uses the Intel® Compute Library for Deep Neural Networks (clDNN) to infer deep neural networks.
It employs for inference execution Intel® Processor Graphics including Intel® HD Graphics and Intel® Iris® Graphics

Before using GPU as OVMS target device, you need to install the required drivers. Next, start the docker container
with additional parameter `--device /dev/dri` to pass the device context and set OVMS parameter `--target_device GPU`.
The command example is listed below :

```
docker run --rm -it --device=/dev/dri -v /opt/model:/opt/model -p 9001:9001 \
ie-serving-py:latest /ie-serving-py/start_server.sh ie_serving model --model_path /opt/model --model_name my_model --port 9001 --target_device GPU
```

### Starting Docker Container using Multi-Device Plugin<a name="multiplugin"></a>

If you have multiple inference devices available (e.g. Myriad VPUs and CPU) you can increase inference throughput by enabling the Multi-Device Plugin. 
With Multi-Device Plugin enabled, inference requests will be load balanced between multiple devices. 
For more detailed information read [OpenVino's Multi-Device plugin documentation](https://docs.openvinotoolkit.org/latest/_docs_IE_DG_supported_plugins_MULTI.html).

In order to use this feature in OpenVINO&trade; Model Server, following steps are required:

Set target_device for the model in configuration json file to MULTI:<DEVICE_1>,<DEVICE_2> (e.g. MULTI:MYRIAD,CPU, order of the devices defines their priority, so MYRIAD devices will be used first in this example)

Below is exemplary config.json setting up Multi-Device Plugin for resnet model, using Intel® Movidius™ Neural Compute Stick and CPU devices:
```json
{"model_config_list": [
   {"config": {
      "name": "resnet",
      "base_path": "/opt/ml/resnet",
      "batch_size": "1",
      "target_device": "MULTI:MYRIAD,CPU"}
   }]
}
```
Starting OpenVINO&trade; Model Server with config.json (placed in ./models/config.json path) defined as above, and with grpc_workers parameter set to match nireq field in config.json:
```
docker run -d  --net=host -u root --privileged --rm -v $(pwd)/models/:/opt/ml:ro -v /dev:/dev -p 9001:9001 \
ovms-py:latest --config_path /opt/ml/config.json --port 9001 
```
Or alternatively, when you are using just a single model, start OpenVINO&trade; Model Server using this command (config.json is not needed in this case):
```
docker run -d  --net=host -u root --privileged --name ie-serving --rm -v $(pwd)/models/:/opt/ml:ro -v \
 /dev:/dev -p 9001:9001 ovms:latest model --model_path /opt/ml/resnet --model_name resnet --port 9001 --target_device 'MULTI:MYRIAD,CPU'
 ```
After these steps, deployed model will perform inference on both Intel® Movidius™ Neural Compute Stick and CPU.
Total throughput will be roughly equal to sum of CPU and Intel® Movidius™ Neural Compute Stick throughput.

### Starting Docker Container Using Heterogeneous Plugin<a name="heteroplugin"></a>

[HETERO plugin](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_supported_plugins_HETERO.html) makes it possible to distribute a single inference processing and model between several AI accelerators.
That way different parts of the DL network can split and executed on optimized devices.
OpenVINO automatically divides the network to optimize the execution.

Similarly to the MULTI plugin, Heterogenous plugin can be configured by using `--target_device` parameter using the pattern: `HETERO:<DEVICE_1>,<DEVICE_2>`.
The order of devices defines their priority. The first one is the primary device while the second is the fallback.<br>
Below is a config example using heterogeneous plugin with GPU as a primary device and CPU as a fallback.

```json
{"model_config_list": [
   {"config": {
      "name": "resnet",
      "base_path": "/opt/ml/resnet",
      "batch_size": "1",
      "target_device": "HETERO:GPU,CPU"}
   }]
}
```


## Security Considerations

OpenVINO Model Server docker containers, by default, starts with the security context of local account ovms with linux uid 5000. It ensure docker container has not elevated permissions on the host machine. This is in line with best practices to use minimal permissions to run docker applications. You can change the security context by adding --user parameter to docker run command. It might be needed for example to load mounted models with restricted access. For example:

```
docker run --rm -d  --user $(id -u):$(id -g)  -v ${pwd}/model/:/model -p 9178:9178 openvino/model_server:latest \
--model_path /model --model_name my_model

``` 
OpenVINO Model Server currently doesn't provide access restrictions and traffic encryption on gRPC and REST API endpoints. The endpoints can be secured using network settings like docker network settings or network firewall on the host. The recommended configuration is to place OpenVINO Model Server behind any reverse proxy component or load balancer, which provides traffic encryption and user authorization.