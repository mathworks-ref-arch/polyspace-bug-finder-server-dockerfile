# Build and Customize a Polyspace Bug Finder or Code Prover Server Docker Image

This repository shows you how to build and customize a Docker image for Polyspace Bug Finder&trade; Server&trade; and Polyspace Code Prover&trade; Server&trade;, using the [MATLAB&reg; Package Manager (*mpm*)](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/MPM.md).

You can use this image as a scalable and reproducible method to run Polyspace analyses in containers.

### Requirements
* A valid Polyspace Bug Finder or Code Prover Server license
* The port and host name of a [Network License Manager](https://www.mathworks.com/help/install/administer-network-licenses.html) or the path of a `network.lic` license file. See [Use Network License Manager](#use-the-network-license-manager). 
* Linux® Operating System
* [Docker](https://docs.docker.com/engine/install/)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

## Get Started

### Get Sources and Build Docker Image
 Use these commands to clone this repository and build the Docker image:
```bash
# Clone this repository to your machine.
git clone https://github.com/mathworks-ref-arch/polyspace-bug-finder-server-dockerfile.git

# Navigate to the downloaded folder.
cd polyspace-bug-finder-server-dockerfile

# Select a base image for the container you want to build, for example, Ubuntu, and copy it to a Dockerfile.
cp Dockerfile.ubuntu22.04 Dockerfile

# Build container with a name and tag of your choice.
docker build . -t polyspace-bug-finder-server:r2024a
```
The `build` command looks for the `Dockerfile` in the current directory and builds an image with the name and tag specified by the `-t` option.<br>
If you list the docker images and filter for `polyspace*` after the `docker build` command completes, you see the polyspace-bug-finder-server image.
```bash
$ docker images -f=reference=polyspace*
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
polyspace-bug-finder-server   r2024a    b8cde78ea9c3   About a minute ago   4.43GB
```


### Run Docker Container
After you build the image, create a file to analyze and a folder where you store the analysis results.
```bash
# Create a source and results folder and make them accessible from the running container
mkdir -p src ps_results
chmod 777 src ps_results
# Create a test file to analyze
echo 'int main(){int a; return a;}'> src/simple_niv.c
```
The following `docker run` command:
* Creates and runs a container from the image that you built in the previous section.
* Sets the environment variable `MLM_LICENSE_FILE`, which enables the container to communicate with the license manager.<br> Specify this variable as `portNumber@hostname`. The variable is mandatory unless you provide the license server information when you build the container.<br>
    See [Specify License Server Information in Build Image](#specify-license-server-information-in-build-image). 
* Maps the current folder to the `/work` folder inside the container. 
* Runs the analysis command (`polyspace-bug-finder -sources...`) from the docker container on the test file you created. 
```bash
# Run the container to analyze the test file.
docker run --rm \
-e MLM_LICENSE_FILE=27000@MyServerName.example.com \ 
-v "$(pwd):/work" polyspace-bug-finder-server:r2024a \ 
 polyspace-bug-finder-server -sources /work/src/simple_niv.c -results-dir /work/ps_results
```


Docker automatically cleans up the container and removes its file system when the container exits (`--rm` option).

## Customize Docker Image
The [Dockerfile](https://github.com/mathworks-ref-arch/polyspace-bug-finder-server-dockerfile/blob/main/Dockerfile) supports these Docker build-time variables.

| Argument Name | Default value | Description |
|---|---|---|
| MATLAB_RELEASE | r2024a | The Polyspace Server release you want to install. Must be lower-case, for example: `r2022b`.|
| [LICENSE_SERVER](#build-an-image-with-license-server-information) | *unset* | The port and hostname of the machine that is running the Network License Manager, using the `port@hostname` syntax. For example: `27000@MyServerName`. |

To customize the docker image, use these arguments with the `docker build` command or edit the default values of these arguments directly in the [Dockerfile](https://github.com/mathworks-ref-arch/polyspace-bug-finder-server-dockerfile/blob/main/Dockerfile).

### Examples:

#### **Specify License Server Information in Build Image**

if you include the [license manager information](#obtain-license-manager-information) when you run the `docker build` command, you do not have to specify the `MLM_LICENSE_FILE` variable when you run the container using the `docker run` command.

```bash
# Build container and specify LICENSE_SERVER variable
docker build -t polyspace-bug-finder-server:r2024a --build-arg LICENSE_SERVER=27000@MyServerName.example.com .

# Run the container without specifying the MLM_LICENSE_FILE variable
docker run --rm \
-v "$(pwd):/work" polyspace-bug-finder-server:r2024a \ 
polyspace-bug-finder-server -sources /work/src/simple_niv.c -results-dir /work/ps_results
```
> **Note**: If you specify the value of the LICENSE_SERVER variable in the Dockerfile, you can omit it when running the `docker build` command.
## Use Network License Manager
The Polyspace Bug Finder Server and Polyspace Code Prover Server products use a concurrent license which requires a Network License Manager to manage license checkouts. 

To enable the container to communicate with the license manager, you must provide the port and hostname of the machine where you installed the license manager or the path to the  `network.lic` file.

### Obtain License Manager Information
Contact your license administrator to obtain the address of your license server, for instance:

 `27000@MyServerName.example.com`, or the path to the `network.lic` file.

You can also get the license manager port and hostname from the `SERVER` line of the `network.lic` file or the `license.dat` file.
```bash
# Sample network.lic
SERVER MyServerName.example.com <optional-mac-address> 27000
USE_SERVER

# Snippet from sample license.dat
SERVER MyServerName.example.com <mac-address> 27000
...
```
### Pass License Manager Information to `build` or `run` Commands
To pass the license manager address (`portNumber@hostname`) to the:
* `docker build` command, assign the address to the [LICENSE_SERVER](#specify-license-server-information-in-build-image) build variable.
* `docker run` command, assign the address to the [MLM_LICENSE_FILE](#run-docker-image) environment variable.

To use the `network.lic` file instead:

1. Copy the file to the same folder as the `Dockerfile`.
1. Uncomment the line `COPY network.lic /opt/matlab/licenses/` in the Dockerfile.

    You do not need to specify the `LICENSE_SERVER` build variable when running the `docker build` command:
    ```bash
    # Example
    docker build -t polyspace-bug-finder-server:r2024a .
    ```
## Remove Docker Image

While docker cleans up containers on exit, the docker image persists in your file system until you explicitely delete it. To delete a docker image, run the `docker image rm` command and specify the name of the container. For instance:
```bash
docker image rm polyspace-bug-finder-server:r2024a
```

---
## More MathWorks Docker Resources
* Explore prebuilt MATLAB Docker Containers on [Docker Hub](https://hub.docker.com/r/mathworks):
    * [MATLAB Containers on Docker Hub](https://hub.docker.com/r/mathworks/matlab) hosts container images for multiple MATLAB releases.
    * [MATLAB Deep Learning Container Docker Hub repository](https://hub.docker.com/r/mathworks/matlab-deep-learning) hosts container images with toolboxes suitable for deep learning.

## Feedback
If you encounter a technical issue or have an enhancement request after trying this repository with your environment, create an issue [here](https://github.com/mathworks-ref-arch/polyspace-bug-finder-server-dockerfile/issues).

----

Copyright (c) 2022-2024 The MathWorks, Inc. All rights reserved.

----
