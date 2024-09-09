---
title: Docker 搭建深度学习环境
date: 2024-09-09 16:25:12
tags:
  - "docker"
excerpt: "通过 Docker 搭建深度学习环境，以及在 Docker 中使用 CUDA。"
categories: "深度学习"
---


# 1.安裝NVIDIA Container Toolkit套件

对于 Arch 系发行版，可以直接通过下面指令安装：
```bash
sudo pacman -S nvidia-container-toolkit
```

然后新增设定：
```bash
sudo mkdir /etc/docker
sudo nano /etc/docker/daemon.json
```

添加：
```txt
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

重启 Docker ：
```bash
sudo systemctl restart docker
```

拉取CUDA 12.1版本的镜像测试：
```bash
sudo docker run --rm --gpus all nvidia/cuda:12.1.0-base-ubuntu20.04 nvidia-smi
```
检查是否能在容器中运行`nvidia-smi`：
![nvidia-smi](https://ivonblog.com/posts/archlinux-install-nvidia-drivers/images/Screenshot_20231226_160611.webp)

# 2.各个 CUDA `tag`的区别

其它 CUDA 镜像可以通过在 DockerHub 找到：[nvidia/cuda](https://hub.docker.com/r/nvidia/cuda)

1. **Base** : Includes only the CUDA runtime (cudart), which is essential for running CUDA-based applications but doesn't come with extra libraries or development tools.
2. **Runtime** : Builds on the `base` image and adds CUDA math libraries like cuBLAS, cuFFT, and cuRAND. This version is suitable for running applications that require these additional libraries without needing to compile CUDA code.
3. **Devel** : Builds on the `runtime` image and includes all the necessary headers and development tools like `nvcc` for compiling and building CUDA applications. This is useful for developers building or extending CUDA applications.
4. **CUDNN/Devel Runtime** : Adds cuDNN libraries to the runtime or devel image for working with deep learning applications.

Different versions also support various operating systems like Ubuntu 20.04, 22.04, or 24.04. The `devel` tags are especially useful when you need to compile custom CUDA code, whereas the `runtime` tags are lighter and designed for deployment【76†source】【77†source】.

# 3.搭建 PyTorch

根据需要拉取`PyTorch`镜像：[pytorch/pytorch](https://hub.docker.com/r/pytorch/pytorch)

我这里以最新版为例：
```bash
docker pull pytorch/pytorch
```

可以先查看下镜像内容（比如端口、挂载点）：
```bash
docker inspect pytorch/pytorch:latest
```
这个镜像的默认挂载点是`/workspace`。
```bash
docker run -it --name=pytorch -v ~/workspace:/workspace -p 8888:8888 pytorch/pytorch:latest
```
这里映射的端口是`jupyter`的。需要在容器里手动安装`jupyter`。
```bash
conda install jupyter
```

运行`jupyter`：
```bash
jupyter lab --ip=0.0.0.0 --no-browser --allow-root
```
启动`Jupyter Lab`时，默认情况下它只绑定到`localhost`，并不会对外开放。所以你需要明确绑定`0.0.0.0`，以便它能从容器外部访问。