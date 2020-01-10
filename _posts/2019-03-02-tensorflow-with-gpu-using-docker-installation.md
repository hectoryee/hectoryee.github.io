---
title: Tensorflow with GPU Install using Docker
excerpt: Docker is the easiest way to enable TensorFlow GPU support on Linux since only the NVIDIA® GPU driver is required on the host machine (the NVIDIA® CUDA® Toolkit does not need to be installed).
published: true
date: 2019-03-02
categories: project
tags: jupyter-notebook tensorflow gpu docker
---

## Installing Tensorflow with GPU Support using Docker
1. [Cuda](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation) installation.
2. Install [nvidia-docker](https://github.com/NVIDIA/nvidia-docker).
3. Check if a GPU is available. 
``` bash
lspci | grep -i nvidia
```
4. Verify your `nvidia-docker` installation.
``` bash
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

    ```
    Sun Mar  3 13:20:01 2019       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 418.39       Driver Version: 418.39       CUDA Version: 10.1     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce 940MX       On   | 00000000:01:00.0 Off |                  N/A |
    | N/A   49C    P8    N/A /  N/A |    380MiB /  4046MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    +-----------------------------------------------------------------------------+
    ```


> Note: `nvidia-docker` v1 uses the `nvidia-docker` alias, where v2 uses `docker --runtime=nvidia`.


### [Build from source](https://www.tensorflow.org/install/source#gpu_support_2)
The following example downloads the TensorFlow `:nightly-devel-gpu-py3` image and uses `nvidia-docker` to run the GPU-enabled container. This development image is configured to build a Python 3 *pip* package with GPU support:
``` bash
docker pull tensorflow/tensorflow:nightly-devel-gpu-py3

docker run --runtime=nvidia -it -w /tensorflow -v $PWD:/mnt -e HOST_PERMS="$(id -u):$(id -g)" \
    tensorflow/tensorflow:nightly-devel-gpu-py3 bash
```

Then, within the container's virtual environment, build the TensorFlow package with GPU support:
``` bash
./configure  # answer prompts or use defaults

bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

./bazel-bin/tensorflow/tools/pip_package/build_pip_package /mnt  # create package

chown $HOST_PERMS /mnt/tensorflow-version-tags.whl
```

Install and verify the package within the container and check for a GPU:
``` bash
pip uninstall tensorflow  # remove current version

pip install /mnt/tensorflow-version-tags.whl
cd /tmp  # don't import from source directory
python -c "import tensorflow as tf; print(tf.contrib.eager.num_gpus())"
```

## Common Issues
- [ CUDA 9.0 unsupported gcc versions later than 6 #731 Closed](https://github.com/ethereum-mining/ethminer/issues/731)
``` bash
sudo apt-get install gcc-6 g++-6
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 10
```

## References
- [tyokota.lab Ubuntu 16.04 + CUDA 9.1 + cuDNN 7.0.5 + Tensorflow 1.4.0](http://tyokota.hatenablog.com/entry/2017/12/20/170451)