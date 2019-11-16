---
title: Install Tensorflow with GPU
excerpt: Install CUDA 10.0 and cuDNN 7.0 for TensorFlow/PyTorch (GPU) on Ubuntu 18.04.
published: true
date: 2019-04-12
categories: project
tags: tensorflow gpu install
---

## Install NVIDIA Graphics Driver 
1.Use NVIDIA proprietary driver. Check installation `nvidia-smi`.

``` bash
Sun Apr 14 15:23:21 2019       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.39       Driver Version: 418.39       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce 940MX       Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   50C    P5    N/A /  N/A |    460MiB /  4046MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1600      G   /usr/lib/xorg/Xorg                           225MiB |
|    0      1758      G   /usr/bin/gnome-shell                         156MiB |
|    0      4577      C   python3                                       24MiB |
|    0      4877      G   ...-token=6290163DAE60CC3BAE4735A7678A09B4    49MiB |
+-----------------------------------------------------------------------------+
```

To install latest driver:
``` bash
sudo add-apt-repository ppa:graphics-drivers/ppa

sudo apt update

sudo apt upgrade

ubuntu-drivers list

sudo apt install nvidia-driver-VERSION_NUMBER_HERE
```



To solve

E: Unable to correct problems, you have held broken packages.

Try:

``` bash
sudo apt-get install --fix-broken xorg-video-abi-11 xserver-xorg-core
```
or
``` bash
sudo apt-get remove --purge nvidia-*
sudo ubuntu-drivers autoinstall
sudo service lightdm restart
```


## Install CUDA Toolkit
2.Download [CUDA installer runfile](https://developer.nvidia.com/cuda-downloads). Check version 
* 10.0.
* 418-10.0
* 390-9.0

3.Stop display manager. `ctrl` + `Alt` + `F3`.

``` bash
# disable the graphical target
systemctl isolate multi-user.target

# unload Nvidia drivers
sudo modprobe -r nvidia-drm
```
4.Execute the runfile installer. Install CUDA Toolkit ans samples.

5.Start graphics.

``` bash
# start graphical env again
systemctl start graphical.target
```
6.After the installation finishes, configure the runtime library.

``` bash
sudo bash -c "echo /usr/local/cuda/lib64/ > /etc/ld.so.conf.d/cuda.conf"
sudo ldconfig
```
7.Append string `/usr/local/cuda/bin` to system file `/etc/environment` so that `nvcc` will be included in `$PATH`.

``` bash
sudo vim /etc/environment
```
Add `:/usr/local/cuda/bin` at the end before the `"`.

8.`reboot`.

### Test installation
9.

``` bash
cd /usr/local/cuda-10.0/samples/
sudo make
```

10.After it completes.

``` bash
cd /usr/local/cuda/samples/bin/x86_64/linux/release
./deviceQuery
```
``` bash
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce 940MX"
  CUDA Driver Version / Runtime Version          10.1 / 10.0
  CUDA Capability Major/Minor version number:    5.0
  Total amount of global memory:                 4046 MBytes (4242604032 bytes)
  ( 4) Multiprocessors, (128) CUDA Cores/MP:     512 CUDA Cores
  GPU Max Clock rate:                            861 MHz (0.86 GHz)
  Memory Clock rate:                             2505 Mhz
  Memory Bus Width:                              64-bit
  L2 Cache Size:                                 1048576 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            No
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.1, CUDA Runtime Version = 10.0, NumDevs = 1
Result = PASS
```

## Install cuDNN 7.0
11.Download from [cuDNN download page](https://developer.nvidia.com/rdp/cudnn-download). Download all three .deb files: runtime lib, developer lib, code samples lib.

12.

``` bash
sudo dpkg -i libcudnn7_7.5.0.56-1+cuda10.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.5.0.56-1+cuda10.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.5.0.56-1+cuda10.0_amd64.deb
```

### Verify installation
13.

``` bash
cp -r /usr/src/cudnn_samples_v7/ ~
cd ~/cudnn_samples_v7/mnistCUDNN
make clean && make
./msnistCUDNN
```

## Configure the CUDA and cuDNN library paths
14.

``` bash
# put the following line in the end or your .bashrc file
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64"

source ~/.bashrc
```

## Install TensorFlow-GPU
15.` pip install --upgrade tensorflow-gpu`.

### Try it out
16.

``` bash
python
>>> import tensorflow as tf
>>> sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
```

``` bash
2019-04-14 15:04:49.833765: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
2019-04-14 15:04:49.858219: I tensorflow/core/platform/profile_utils/cpu_utils.cc:94] CPU Frequency: 2400000000 Hz
2019-04-14 15:04:49.858464: I tensorflow/compiler/xla/service/service.cc:150] XLA service 0x5620497c66b0 executing computations on platform Host. Devices:
2019-04-14 15:04:49.858487: I tensorflow/compiler/xla/service/service.cc:158]   StreamExecutor device (0): <undefined>, <undefined>
2019-04-14 15:04:49.984415: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:998] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2019-04-14 15:04:49.985380: I tensorflow/compiler/xla/service/service.cc:150] XLA service 0x56204907b920 executing computations on platform CUDA. Devices:
2019-04-14 15:04:49.985407: I tensorflow/compiler/xla/service/service.cc:158]   StreamExecutor device (0): GeForce 940MX, Compute Capability 5.0
2019-04-14 15:04:49.985716: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1433] Found device 0 with properties: 
name: GeForce 940MX major: 5 minor: 0 memoryClockRate(GHz): 0.8605
pciBusID: 0000:01:00.0
totalMemory: 3.95GiB freeMemory: 3.61GiB
2019-04-14 15:04:49.985739: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1512] Adding visible gpu devices: 0
2019-04-14 15:04:49.987838: I tensorflow/core/common_runtime/gpu/gpu_device.cc:984] Device interconnect StreamExecutor with strength 1 edge matrix:
2019-04-14 15:04:49.987862: I tensorflow/core/common_runtime/gpu/gpu_device.cc:990]      0 
2019-04-14 15:04:49.987876: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1003] 0:   N 
2019-04-14 15:04:49.988147: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1115] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 3401 MB memory) -> physical GPU (device: 0, name: GeForce 940MX, pci bus id: 0000:01:00.0, compute capability: 5.0)
Device mapping:
/job:localhost/replica:0/task:0/device:XLA_CPU:0 -> device: XLA_CPU device
/job:localhost/replica:0/task:0/device:XLA_GPU:0 -> device: XLA_GPU device
/job:localhost/replica:0/task:0/device:GPU:0 -> device: 0, name: GeForce 940MX, pci bus id: 0000:01:00.0, compute capability: 5.0
2019-04-14 15:04:49.989223: I tensorflow/core/common_runtime/direct_session.cc:317] Device mapping:
/job:localhost/replica:0/task:0/device:XLA_CPU:0 -> device: XLA_CPU device
/job:localhost/replica:0/task:0/device:XLA_GPU:0 -> device: XLA_GPU device
/job:localhost/replica:0/task:0/device:GPU:0 -> device: 0, name: GeForce 940MX, pci bus id: 0000:01:00.0, compute capability: 5.0
```

## References
- [Tensorflow steps](https://www.tensorflow.org/install/gpu)
- [Install CUDA 9.0 and cuDNN 7.0 for TensorFlow/PyTorch (GPU) on Ubuntu 16.04](https://medium.com/@zhanwenchen/install-cuda-and-cudnn-for-tensorflow-gpu-on-ubuntu-79306e4ac04e)
- [StackExchange: How to unload kernel module 'nvidia-drm'?](https://unix.stackexchange.com/questions/440840/how-to-unload-kernel-module-nvidia-drm)
