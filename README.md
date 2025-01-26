# DevOps Directive Docker Course

This is the companion repo to: [Complete Docker Course - From BEGINNER to PRO! (Learn Containers)](https://youtu.be/RqTEHSBrYFw)

[![](./readme-assets/thumbnail.jpg)](https://youtu.be/RqTEHSBrYFw)

## Notes by Manjunadh

**Docker Commands CheatSheet** is available [here](./readme-assets/docker_cheatsheet.pdf)

Before virtualization was invented, all programs ran directly on the host system (called bare-metal approach) and this works as follows:

![Baremetal Image](./01-history-and-motivation/readme-assets/bare-metal.jpg)

Assume we have a ubuntu system that is used to serve the app.The following is the description of each layer:
- Physical System: The actual Intel chip and RTX4090 GPU hardware
- Operating System: Ubuntu installed on the hardware
- Binaries/Libraries: Conda environment/Python
- Application: The actual project code for serving the app.

#### Virtual Machines: 
Virtual machines use a system called a "hypervisor" that can carve up the host resources into multiple isolated virtual hardware configuration which you can then treat as their own systems (each with an OS, binaries/libraries, and applications).

Hypervisors can be of 2 types:
- Type1 Hypervisor: The hypervisors used in cloud servers like AWS, GCP etc. that directly run an OS on the physical hardware.
- Type2 Hypervisor: The hypervisors used in Windows like "Oracle's Virtual Box" come under this category. These are softwares run on top of an existing OS, say Ubuntu and MacOS images running in Virtual Box which inturn runs on say Windows installation.
This is described as shown below:

![](./01-history-and-motivation/readme-assets/hypervisor-types.jpg)

The complete architecture is as follows:
![](./01-history-and-motivation/readme-assets/virtual-machine.jpg)

This Virtualisation architecture is very clean but spinning up a virtual machine that has an OS will make this slow. It might be faster than the traditional approach but it is still slow.

Idea is to offload even the OS so that virtualisation only spins up deployed app related resources and code.

#### Container:
Containers are similar to virtual machines in that they provide an isolated environment for installing and configuring binaries/libraries, but rather than virtualizing at the hardware layer containers use native linux features (cgroups + namespaces) to provide that isolation while still sharing the same kernel.

#### Docker Installation:
* Install Docker on Linux(https://docs.docker.com/engine/install/ubuntu/)
* If the system is an old legacy system, install Docker Toolbox instead(not relevant for me)
* Dockerhub is the container registry hosted by Docker and is the container equivalent to github.
* alpine is a docker image that is commonly used in tutorial videos. This is an image of Alpine Linux, one of Linux many distros that occupies only 5MB memory space.

#### Container Structure

**Docker Container Image(also called a docker image)** is a standalone, executable file used to create a container.

When we create a container from a container image, everything in the image is treated as read-only, and there is a new layer overlayed on top that is read/write. 

**BaseImage** layer is the virtual OS layer while the other **Build Layers** follow each command in the container image.
This is illustrated as shown below:

![](./04-using-3rd-party-containers/readme-assets/container-filesystem.jpg)

To create a docker image with ubuntu22.04 as the OS:
```bash
docker run --interactive --tty --rm ubuntu:22.04
```
In this command, ```--interactive``` and ```--tty``` are used to give an interactive bash shell(if interactive tag is not used, the shell is still created but can't read any inputs) while ```--rm``` specifies that any changes done in this container are lost as soon as the container is shutdown. This is because any changes done to the container say installing wget for example is done on the Container Layer which is read-write in nature and once you shutdown the container, that layer is immediately lost. Restarting the container spins up a container with a different container ID.

#### Persisting Data Produced by the Application

Volumes and mounts allow us to specify a location where data should persist beyond the lifecycle of a single container.

- **Volume** is a directory on the host machine that is accessible to the container. Data stored in volume persists even if the container is destroyed.
- Volume data is stored on the filesystem on the host, **but in order to interact with the data in the volume, you must mount the volume to a container**. 
***Directly accessing or interacting with the volume data is unsupported, undefined behavior, and may result in the volume or its data breaking in unexpected ways.***

**Mount** is a directory on the host machine that is mounted as a directory in the container. If the container is deleted, the data is erased in the directory on the host machine.

The data can live in a location managed by Docker ```(volume mount)```, a location in your host filesystem ```(bind mount)```, or in memory (```tmpfs mount```, not pictured). This is depicted below:

![](./04-using-3rd-party-containers/readme-assets/volumes.jpg)

NOTE: 
- This third option ***(tmpfs mount) does not persist the data after the container exits***, and is instead used as a temporary store for data you specifically DON'T want to persist (for example credential files). It is included here for completeness but should not be used for application data you want to persist.
- If the container is not deleted and we make changes in the host machine's directory these changes will be reflected in the container's directory.
- Docker mounts are the goto choice if the data is created while testing an app(environment files, app logs etc.) while docker volumes are recommended if your container is say training an AI model and the dataset needs to be stored.

To create a container that retains the data:
```bash
# Create a container from the ubuntu image (with a name and WITHOUT the --rm flag; -it is shortform for --interactive)
docker run -it --name my-ubuntu-container ubuntu:22.04

# Just update the terminal before proceeding
apt update

# Install wget
apt install wget --yes

# Download yolov9-m model checkpoint (Using the command)
wget https://github.com/WongKinYiu/yolov9/releases/download/v0.1/yolov9-m.pt

# To exit and stop the container(use exit command or Ctrl+D)
exit
```

To create a volume mount and attach it
```bash
# create a named volume (my-volume is a directory in the host machine and has its path)
docker volume create my-volume

# Create a container and mounts the volume located at my-volume in host machine at /my-data/ inside the container
docker run  -it --rm --mount source=my-volume,destination=/my-data/ ubuntu:22.04
# There is a similar (but shorter) syntax using -v which accomplishes the same
docker run  -it --rm -v my-volume:/my-data ubuntu:22.04

# Now we can create and store the file into the location we mounted the volume
echo "Hello from the container!" > /my-data/hello.txt
cat my-data/hello.txt


exit
```




## Sponsor

[![](./readme-assets/shipyard-logo.png)](https://shipyard.build/)

Thank you to [Shipyard](https://shipyard.build/) for sponsoring this course! It is because of their support that I am able to provide it to the community free of charge!

Shipyard is the easiest way to generate on demand ephemeral environments (aka a new environment for every pull request). Sign up today at https://shipyard.social/DevOpsDirectivePromo! The first 300 people to use the promo code "DEVOPSDIRECTIVE" will receive an additional 30 days free on either their startup or business tier plans!

## [01 - History and Motivation](01-history-and-motivation/README.md)

Examines the evolution of virtualization technologies from bare metal, virtual machines, and containers and the tradeoffs between them.

## [02 - Technology Overview](02-technology-overview/README.md)

Explores the three core Linux features that enable containers to function (cgroups, namespaces, and union filesystems), as well as the architecture of the Docker components.

## [03 - Installation and Set Up](03-installation-and-set-up/README.md)

Covers the steps to install and configure Docker Desktop on your system.

## [04 - Using 3rd Party Containers](04-using-3rd-party-containers/README.md)

Before we build our own container images, we can familiarize ourselves with the technology by using publicly available container images. This section covers the nuances of data persistence with containers and then highlights some key use cases for using public container images.

## [05 - Example Web Application](05-example-web-application/README.md)

Learning about containerization is interesting, but without a practical example it isn't very useful. In this section we create a 3 tier web application with a React front end client, two apis (node.js + golang), and a database. The application is as simple as possible while still providing a realistic microservice system to containerize.

## [06 - Building Container Images](06-building-container-images/README.md)

Demonstrates how to write Dockerfiles and build container images for the components of the example web app. Starting with a naive implementation, we then iterate towards a production ready container image.

## [07 - Container Registries](07-container-registries/README.md)

Explains what container registries are and how to use them to share and distribute container images.

## [08 - Running Containers](08-running-containers/README.md)

Using the containerized web application from sections 05 and 06, we craft the necessary commands to run our application with Docker and Docker Compose. We also cover the variety of runtime configuration options and when to use them.

## [09 - Container Security](09-container-security/README.md)

Highlights best practices for container image and container runtime security.

## [10 - Interacting with Docker Objects](10-interacting-with-docker-objects/README.md)

Describes how to use Docker to interact with containers, container images, volumes, and networks.

## [11 - Development Workflows](11-development-workflow/README.md)

Establishes tooling and configuration to enable improved developer experience when working with containers.

## [12 - Deploying Containers](12-deploying-containers/README.md)

Demonstrates deploying container applications to production using three different approaches: railway.app, a single node Docker Swarm, and a Kubernetes cluster.
