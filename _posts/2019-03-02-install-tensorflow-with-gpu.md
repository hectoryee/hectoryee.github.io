---
title: Install Tensorflow with GPU
excerpt: 
published: true
date: 2019-03-02
categories: project
tags: notebook tensorflow
---

###  CUDA 9.0 unsupported gcc versions later than 6 #731 
- [ CUDA 9.0 unsupported gcc versions later than 6 #731 Closed](https://github.com/ethereum-mining/ethminer/issues/731)
``` bash
 sudo apt-get install gcc-6 g++-6
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 10
```


## Installing Tensorflow with GPU Support using Docker
- [Build from source](https://www.tensorflow.org/install/source#gpu_support_2)
- []()


### Common Issues
- [How to fix docker: Got permission denied issue](https://stackoverflow.com/questions/48957195/how-to-fix-docker-got-permission-denied-issue)
- Create the docker group.
    ```sudo groupadd docker```
- Add your user to the docker group.
    ```sudo usermod -aG docker $USER```
- Logout and login again and run.
    ```docker run hello-world```

### Running Docker
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

- [A Docker Cheat Sheet](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)