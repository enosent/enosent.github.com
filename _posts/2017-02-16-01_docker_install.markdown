---
layout: post
title:  "Docker + Nvidia-Docker 설치 및 제거"
date:   2017-02-19 12:14:00 +0900
categories: docker
---

## Docker

### * Docker + Tensorflow GPU 테스트 환경을 위해 적용
---

#### 개발환경
  - Ubuntu 16.04 LTS
  - Docker 1.13
  - Tensorflow Ubuntu GPU version v0.12
  - CUDA v8.0

#### Nvidia-Docker
  - GPU를 사용할 수 있도록 지원. Docker 내에 있는 tensorflow에서 GPU 리소스를 사용해서 연산 수행 가능
  - linux kernel 3.10 이상, Docker 1.9 이상에서만 설치 가능

---

#### Docker 설치

```sh
$ sudo apt-get update
$ sudo apt-get install docker-engine
```

기존에 Docker가 설치된 경우에는 `sudo apt-get purge docker-engine`으로 삭제 후 재설치
- purge를 이용해 제거시 기존의 container / image 는 유지됨

---
> E: Unable to locate package docker-engine

docker-engine 패키지를 찾을 수 없는 경우 아래 명령어 실행

```bash
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list

$ sudo apt-get update

$ sudo apt-get install -y docker-engine
```
---

sudo 명령어 없이 실행 가능하도록 권한 설정

```bash
$ sudo usermod -aG docker $(whoami)
```
---

#### Nvidia-Docker 설치

https://github.com/NVIDIA/nvidia-docker

```bash
# Install nvidia-docker and nvidia-docker-plugin
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.0/nvidia-docker_1.0.0-1_amd64.deb
sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb

# Test nvidia-smi
nvidia-docker run --rm nvidia/cuda nvidia-smi
```

#### Nvidia-docker 테스트

```
$ nvidia-docker run --rm nvidia nvidia-smi

+------------------------------------------------------+
| NVIDIA-SMI 340.101    Driver Version: 340.101        |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GT 730      Off  | 0000:01:00.0     N/A |                  N/A |
| 33%   38C    P8    N/A /  N/A |      4MiB /  1023MiB |     N/A      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Compute processes:                                               GPU Memory |
|  GPU       PID  Process name                                     Usage      |
|=============================================================================|
|    0            Not Supported                                               |
+-----------------------------------------------------------------------------+
```

---
> nvidia-docker | 2017/02/08 14:01:45 Error: unsupported CUDA version: driver 6.5 < image 8.0

버전이 안맞아서 발생. 8.0을 요구하나 설치된 버전이 6.5라서 실행이 안됨.

##### 기존 설치된 CUDA 및 NVIDIA 그래픽 드라이버 제거
- runfile을 이용하면 드라이버를 알아서 설치해주므로 같이 제거

```bash
$ sudo apt-get remove --purge nvidia-*
$ sudo apt-get install ubuntu-desktop
$ sudo rm /etc/X11/xorg.conf
$ sudo nvidia-uninstall

# Reboot
$ sudo shutdown -r now
```

https://developer.nvidia.com/cuda-downloads

##### cuda_8.0.44_linux-run 파일 다운로드
- CUDA 사용을 위한 패키지 설치

```bash
$ sudo apt-get install -y opencl-headers build-essential protobuf-compiler \
   libprotoc-dev libboost-all-dev libleveldb-dev hdf5-tools libhdf5-serial-dev \
   libopencv-core-dev libopencv-highgui-dev libsnappy-dev libsnappy1 \
   libatlas-base-dev cmake libstdc++6-4.8-dbg libgoogle-glog0 libgoogle-glog-dev \
   libgflags-dev liblmdb-dev git python-pip gfortran
$ sudo apt-get clean
$ sudo apt-get install -y linux-image-extra-`uname -r` linux-headers-`uname -r` linux-image-`uname -r`
```

* ```sudo sh cuda_8.0.44_linux.run``` 설치 가이드에 따라 실행
  - Driver를 제거한 상태이므로 설치 필요

```
...
Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 361.77? (y)es/(n)o/(q)uit: y
Install the CUDA 8.0 Toolkit? (y)es/(n)o/(q)uit: y
...
```

##### 환경 구성 ( .bashrc 파일에 추가 )
```
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:/usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH
export PATH=/usr/local/cuda-8.0/bin:$PATH
export CUDA_HOME=/usr/local/cuda
```

##### 설치 확인
```bash
$ nvcc --version

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2016 NVIDIA Corporation
Built on Sun_Sep__4_22:14:01_CDT_2016
```

##### CUDA Test
```bash
$ cd NVIDIA_CUDA-8.0_Samples/1_Utilities/bandwidthTest/
$ make
$ ./bandwidthTest

...
Result = PASS

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.
```

---

#### Docker 제거
- Docker 패키지만 삭제

```
$ sudo apt-get purge docker-engine
```

- Docker 패키지 + 의존성 패키지 삭제

```
$  sudo apt-get remove --auto-remove docker-engine
```

- 컨테이너, 볼륨, 설정, 이미지 파일 제거

```
$ rm -rf /var/lib/docker
```

---

#### apt-get update 시 아래 오류로 실패한 경우

> E: The method driver /usr/lib/apt/methods/http could not be found.

```$ sudo apt-get install apt-transport-https``` 실행하면 해결
