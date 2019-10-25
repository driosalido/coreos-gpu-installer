# coreos-gpu-installer
This repository contains scripts to build Docker containers that can be used to download, compile and install Nvidia GPU drivers on CoreOS Container Linux OS images.

## How it works

This docker container is meant to be run in every coreo-os server that has `nvidia` GPUS. 

It is based on [coreos-gpu-installer](https://github.com/shelmangroup/coreos-gpu-installer). It compiles the driver and the modules and install the files in the host using a overlay filesystem. The drivers are built on the fly every time the server is booted. 

It can be run as a deamonset inside kubernetes or via a systemd unit on every server.

The desired Nvidia Driver version can be assigned using `NVIDIA_DRIVER_VERSION` env variable (Default `410.78`)

## How to use

### Inside Kubernetes

Example command:
``` shell
kubectl create -f daemonset.yaml
```

### Systemd Unit

```
[Unit]
Description=NVIDIA driver
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=20m
ExecStartPre=-/usr/bin/docker rm nvidia-driver
ExecStartPre=/bin/sh -c 'while ! /usr/bin/docker pull srcd/coreos-nvidia-driver-installer:latest; do sleep 1; done'
ExecStartPre=/usr/bin/docker run --rm \
  -v /:/root \
  -v /dev:/dev \
  -v /opt/nvidia:/usr/local/nvidia \
  --privileged \
  -e NVIDIA_INSTALL_DIR_HOST=/opt/nvidia \
  -e NVIDIA_INSTALL_DIR_CONTAINER=/usr/local/nvidia \
  -e ROOT_MOUNT_DIR=/root \
  srcd/coreos-nvidia-driver-installer:latest
ExecStart=/usr/bin/docker run --rm --name nvidia-driver srcd/coreos-nvidia-driver-installer:latest sleep infinity  
ExecStop=/usr/bin/docker stop nvidia-driver

[Install]
WantedBy=multi-user.target
```