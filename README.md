# vllm-benchmark-droplet-gpu
Repo to setup a docker image of various VLLMs on a Droplet GPU and benchmark it


## Prerequisites:
- Once you have created a GPU Droplet you want by contacting/requesting from [here](https://www.digitalocean.com/products/gpu-droplets)
- check `nvidia-smi`  
- Create a new user (say `dev`) just for good measure, its not best practice to use root for everything
```
sudo adduser dev 
<!-- Provide a password -->

sudo usermod -aG sudo dev

# swtich from root to dev user:
su - dev
```

- [optional] Create a working dir for the project
``` 
mkdir work_dir
cd work_dir
```

## Setup Machine
- In order for us to use the machine properly, we have to install a bunch of packages, example: nvidia drivers, cuda toolkit, docker etc.
- It takes roughly 10 minutes to setup. And might require a reboot. 
1. Create some Env variables:
```
export APT_INSTALL="apt-get install -y --no-install-recommends"
export PIP_INSTALL="python3 -m pip --no-cache-dir install --upgrade"
export GIT_CLONE="git clone --depth 10"
``` 

2. Install common pkgs (many will be already installed):
```
sudo apt-get update && \
        sudo $APT_INSTALL \
        gcc \
        make \
        pkg-config \
        apt-transport-https \
        build-essential \
        apt-utils \
        ca-certificates \
        wget \
        rsync \
        git \
        vim \
        mlocate \
        libssl-dev \
        curl \
        openssh-client \
        unzip \
        unrar \
        zip \
        awscli \
        csvkit \
        emacs \
        joe \
        jq \
        dialog \
        man-db \
        manpages \
        manpages-dev \
        manpages-posix \
        manpages-posix-dev \
        nano \
        iputils-ping \
        sudo \
        ffmpeg \
        libsm6 \
        libxext6 \
        libboost-all-dev \
        gnupg \
        cifs-utils \
        zlib1g \
        software-properties-common
```

3. Install docker.io:
```
sudo apt install docker.io
```

4. Install nvidia docker: (this allows the docker images to use the host GPUs)
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install the NVIDIA docker toolkit
sudo apt-get update
sudo apt-get install -y nvidia-docker2

sudo systemctl restart docker
```

5. Quick test to see if the nvidia-docker is working:   `sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi` should return all the GPUs
```
Thu Aug 22 18:21:40 2024       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.183.01             Driver Version: 535.183.01   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA H100 80GB HBM3          On  | 00000000:00:0A.0 Off |                    0 |
| N/A   30C    P0              68W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   1  NVIDIA H100 80GB HBM3          On  | 00000000:00:0B.0 Off |                    0 |
| N/A   30C    P0              70W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   2  NVIDIA H100 80GB HBM3          On  | 00000000:00:0C.0 Off |                    0 |
| N/A   27C    P0              68W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   3  NVIDIA H100 80GB HBM3          On  | 00000000:00:0D.0 Off |                    0 |
| N/A   29C    P0              69W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   4  NVIDIA H100 80GB HBM3          On  | 00000000:00:0E.0 Off |                    0 |
| N/A   30C    P0              71W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   5  NVIDIA H100 80GB HBM3          On  | 00000000:00:0F.0 Off |                    0 |
| N/A   29C    P0              69W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   6  NVIDIA H100 80GB HBM3          On  | 00000000:00:10.0 Off |                    0 |
| N/A   29C    P0              69W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
|   7  NVIDIA H100 80GB HBM3          On  | 00000000:00:11.0 Off |                    0 |
| N/A   27C    P0              69W / 700W |      0MiB / 81559MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```

## Run Docker Image
- Pull the docker image from the [vllm docker hub](https://hub.docker.com/r/vllm/vllm-openai/tags) and run it as following:
```
sudo docker run --runtime nvidia --gpus all \
    -v ~/.cache/huggingface:/root/.cache/huggingface \
    --env "HUGGING_FACE_HUB_TOKEN=<secret>" \
    -p 8000:8000 \
    --ipc=host \
    vllm/vllm-openai:latest \
    --model mistralai/Mistral-7B-v0.1
```