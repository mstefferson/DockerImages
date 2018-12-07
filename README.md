# DockerImages

My baseline docker images for machine learning project (CPU/GPU), flask apps, and (hopefully) more. Feel free to pull or build the image of your choice. However, you may want to see the example on how to build a project on top of one of these images.

## Pre-requests
* [Docker](https://docs.docker.com/install/)
* [CUDA](https://developer.nvidia.com/cuda-downloads) drivers* (GPU)
* [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) (GPU)

*Note, I have never install the CUDA driver before as I normally use an instance that already has them installed (see below).

If you're doing deep learning, you should be using a GPU---you're time is worth more than the $0.50 and hour to run on a GPU! The two main options are AWS and paperspace. I ran into some issues on paperspace so I've included some notes before. I haven't tried on AWS yet, but I will fill out some notes if I run into problems.


## Install

### From docker hub
#### ml_bot: go to for machine and deep learning
```
docker pull mstefferson/ml_bot:latest
```

#### flask_bot: go to for lightweight flask apps
```
docker pull mstefferson/flask_bot:latest
```

### Build from source
```
git clone https://github.com/mstefferson/DockerImages
```

Go into the directory of the image you want to build. _E.g_, if you want to build the ml_bot,
```
cd ml_bot
```

Then build the image
```
./build
```

## Image info
Here is some basic info on what comes with each image

#### ml_bot
I wanted to build an image that had all of the main tools that could be ran on a cpu or gpu. Is it necessary to have both Pytorch and Keras if you're only using one? Of course not! I have been bouncing between both tools, and I just wanted to have a complete image that I could use for various things that can be ran on a CPU/GPU! I made build individual images if there is interest. Here's what you get:
* PyTorch
* Tensorflow
* Keras
* sklearn
* Pandas
* Jupyter notebooks
* pytest and coverage

#### flask_bot
A lightweight image for building flask apps. What you get:
* flask
* pytest and coverage

## How to run an image
Within each directory, there is a run_docker_* command. Feel free to use this as an example to get up and running. This is how I like to run my docker images. Note, you can do `docker run` from any directory in the shell. Here is an example with an explanation of what each piece is doing:

```
sudo docker run --rm -ti -v ${PWD}:/app/ -p 8989:8989 --hostname localhost --ipc=host mstefferson/ml_bot:latest bash
```
* `docker run`: run a container (a computer!) from the docker image
* `--rm`: when you exit the container, remove it
* `-it`: interative mode. The container can be running in the background and you can execute a command from the host command line, like open a bash shell, to do be executed on the container.
* `-v ${PWD}:/app/`: mount the current WD into the directory `/app` on the container, which is the WD of the container.
* `-p 8989:8989`: map port 8989 on the host machine to 8989 on the container
* `--hostname localhost`: call the hostname localhost. This is for notebooks
* `--ipc=host`: errr... honestly I do not fully understand this. I believe it allows the container and host (your machine) to talk to each other.
* `mstefferson/ml_bot:latest`: image I want to run
* `bash`: run a bash shell!

If you want to run a jupyter notebook, I gave an example executable, which should be ran inside the docker image

```
./run_jn
```

## Addition install info for cloud computing

### Paperspace

To run GPU docker images, you need CUDA drivers, docker, and nvidia-docker. Below are the steps to get set-up. Paperspace comes with some of these, but they weren't up to date as of 20181125. Here are notes (sources linked) on how to get the most updated versions of [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/#set-up-the-repository) and [nvidia docker](https://github.com/NVIDIA/nvidia-docker) to run on Paperspace.

#### Get ML-in-a-box instance time (CUDA drivers)

ML-in-a-box instance has the CUDA drivers installed so it's easiest to just select that.

Under choose OS, select Public Templates-> ML-in-a-Box (Ubuntu 16.04)

Yes, 18.04 is available and should also work, but I am always skeptical for new releases!

Choose your storage (100 Gb sufficed for me) and GPU (P4000 for me). I turned off snapshot (I believe it's extra), and I set-up a public IP so  can ssh into the machine.

#### Update docker
ML-in-box comes with docker installed, but I was having version issues when installing nvidia-docker.  I stole these notes from A fix is to update docker:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
sudo apt-get update
sudo apt-get install docker-ce
```

#### Update nvidia-docker
Nvidia docker 1.# is also pre-installed, but we should update it to 2.#. Feel free to run the entire block or do it in pieces (between comments) to make sure all steps work

```
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
sudo docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```

### AWS
I don't like Amazon so I haven't the set-up on there yet. Should be straight forward it you pick a GPU instance with a reasonable template.


## TODO

* Streamlit
* Docker compose?
