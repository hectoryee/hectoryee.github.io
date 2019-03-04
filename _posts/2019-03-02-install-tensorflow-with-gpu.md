---
title: Install Tensorflow with GPU
excerpt: 
published: true
date: 2019-03-02
categories: project
tags: notebook tensorflow
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

My running version:
``` bash
docker run --runtime=nvidia -it -w /notebooks -p 8888:8888 -v $(realpath ~/project/):/notebooks -e HOST_PERMS="$(id -u):$(id -g)"     tensorflow/tensorflow:nightly-devel-gpu-py3 bash
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

### Common Issues
- [ CUDA 9.0 unsupported gcc versions later than 6 #731 Closed](https://github.com/ethereum-mining/ethminer/issues/731)
``` bash
sudo apt-get install gcc-6 g++-6
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 10
```
- [How to fix docker: Got permission denied issue](https://stackoverflow.com/questions/48957195/how-to-fix-docker-got-permission-denied-issue)
    - Create the docker group.
    ```sudo groupadd docker```
    - Add your user to the docker group.
    ```sudo usermod -aG docker $USER```
    - Logout and login again and run.
    ```docker run hello-world```



## [Using a Jupyter Notebook within a Docker Container](https://devtalk.nvidia.com/default/topic/1032202/docker-and-nvidia-docker/using-a-jupyter-notebook-within-a-docker-container/)
``` bash
docker run --runtime=nvidia -it -v "/my-local-computer-files/:/my-docker-container/" my-nvidia-container
```

### Add flag
- `-p 8888:8888`
- `-v "/my-local-computer-files/:/my-docker-container/"`

Example
``` bash
docker run --runtime=nvidia -it -p "8888:8888" -v "/my-local-computer-files/:/my-docker-container/" my-nvidia-container
```

Inside docker run:
``` bash
jupyter notebook --port=8888 --ip=0.0.0.0 --allow-root --no-browser .
```


## [Running Docker](https://stackify.com/docker-tutorial/)
``` bash
# Run a Docker image
docker run hello-world

# List images
docker image ls

# List the containers on our system
docker ps -a

# List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

# Reuse a container
docker start -attach <container name>

# Look inside the container and show the running processes
docker top <container name>

# Stop container
docker stop <container name>

# Remove docker
docker rm <container name>

```
- `-attach` tells Docker to connect to the container output so we can see the results
- `-it` flags allow us to interact with the shell.

### Share system resources with a container
``` bash
docker run -v /full/path/to/html/directory:/usr/share/nginx/html:ro -p 8080:80 -d nginx
```
- `-v /full/path/to/html/directory:/usr/share/nginx/html:ro` maps the directory holding our web page to the required location in the image. The ro field instructs Docker to mount it in read-only mode. It’s best to pass Docker the full paths when specifying host directories.
- `-p 8080:80` maps network service port 80 in the container to 8080 on our host system.
- `-d` detaches the container from our command line session. Unlike our previous two examples, we don’t want to interact with this container.
- `nginx` is the name of the image.

Able to reach the web server on localhost:8080.

1814
12-1839
22-1905

- [Cleaning Docker containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)

## References
- [tyokota.lab Ubuntu 16.04 + CUDA 9.1 + cuDNN 7.0.5 + Tensorflow 1.4.0](http://tyokota.hatenablog.com/entry/2017/12/20/170451)