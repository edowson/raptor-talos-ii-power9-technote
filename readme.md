# Raptor Talos II Secure Workstation Setup Guide

This repository provides a comprehensive guide for setting up a Raptor Talos II Secure Workstation for AI, machine learning and robotics software development.

This system's motherboard has a dual-socket configuration and can support 4, 8, 18 and 22-core IBM Power9 CPUs.

## Installation checklist

### Operating system install

[Install Ubuntu-18.04 LTS on a Raptor Talos II Power9 system](./ubuntu/ubuntu-18.04-install-power9-raptor-talos-ii.md):
- [x] Ubuntu-18.04 LTS for ppc64el, server, ubuntu-desktop.
- [x] NVIDIA proprietary graphics driver v418.67 for ppc64el, tested with the following GPUs:

  - [x] Tesla V100 with 16GB HBM2
  - [ ] Titan-V with 12GB HBM2
  - [ ] Quadro RTX 8000 with 48GB GDDR6
  - [x] GeForce RTX 2070 with 8GB GDDR6
  - [ ] Quadro P6000 with 24GB GDDR5


### Graphics and acceleration libraries.

[Install NVIDIA graphics and acceleration libraries](./ubuntu-18.04-install-power9-raptor-talos-ii-nvidia-graphics-cuda-cudnn-tensorrt.md):
- [x] NVIDIA proprietary graphics driver 418.87
- [x] CUDA-10.1
- [ ] cuDNN-7.6.2.24
- [ ] TensorRT-5.1.5.0
- [ ] NCCL-2.4.7

### Remote desktop access.

[Compile libjpeg-turbo, VirtualGL and TurboVNC](./ubuntu/ubuntu-18.04-compile-libjpeg-turbo-virtualgl-turbovnc.md) from sources:
- [x] libjpeg-turbo v2.0.2, compiled from source, branch: `2.0.2`, commit id: `a4aa30d9a080bbc50421285049e4379dcaf8a669`
- [x] VirtualGL v2.6.3, compiled from source, branch: `master`, commit id: `b74c7b86f21703b201682e2078e1188fc46c3b1f`
- [x] TurboVNC v2.2.2, compiled from source, branch: `master`, commit id: `8cf390455d62dfd50595f9edfc53bc1349a4c481`

Choose any **one** of the following options to configuring remote desktop access:
- [Configure Ubuntu-18.04 remote desktop access using VirtualGL and TurboVNC using gdm3 display manager](./ubuntu/ubuntu-18.04-configure-virtualgl-turbovnc-gdm3.md) (preferred).
- [Configure Ubuntu-18.04 remote desktop access using VirtualGL and TurboVNC using lightdm display manager](./ubuntu/ubuntu-18.04-configure-virtualgl-turbovnc-lightdm.md).

### Common development libraries.

- [x] GoogleTest, built from source, branch: `master`, commit id: `90a443f9c2437ca8a682a1ac625eba64e1d74a8a`.

### Common development tools.

- [x] CLion-2019.2 with OpenJDK 11.0.4 2019-07-16.
- [ ] PyCharm-2019.2 with OpenJDK 11.0.4 2019-07-16.

### DevOps.

- [ ] docker-ce on ppc64el.
- [ ] nvidia-docker2 on ppc64el, verify nvidia gpu access from docker container on ppc64el.
- [ ] kubernetes v1.14.3 on ppc64el.
- [ ] grafana dashboard.
- [ ] kubeflow.

### Game engines.

- [ ] UE4.22.3 port to ppc64le.
- [x] UE4.23.1 port to ppc64le.

### Machine learning libraries.

Machine learning libraries:
- [ ] TensorFlow-2.0.0-beta1, compiled from source, branch: `2.0.0`. commit id: ``
- [ ] PyTorch-1.2.0, compiled from source, branch: `v1.2.0`. commit id: ``

### Robotics libraries.

- [ ] ROS1 Melodic Morena, compile from source for ppc64el.
- [ ] ROS2 Dashing Diademata, compile from source for ppc64el.

### Robotics simulation environments and software development kits.

- [ ] OpenAI Gym
- [ ] NVIDIA Isaac SDK, UE4.22.3 sim.
