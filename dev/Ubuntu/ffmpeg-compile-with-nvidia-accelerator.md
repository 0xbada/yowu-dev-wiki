---
title: WSL2 Ubuntu에서 ffmpeg 컴파일 (with Nvidia Accelerator)
description: 
published: true
date: 2023-02-19T11:04:04.971Z
tags: ffmpeg, nvidia, cuda
editor: markdown
dateCreated: 2023-02-19T11:04:04.971Z
---

> - WSL2 Ubuntu 환경에서 ffmpeg 를 사용할 일이 생김
> - APT 저장소의 ffmpeg는 영상 처리에 CPU를 사용한다
> - ffmpeg가 Nvidia 그래픽카드의 하드웨어 가속을 사용하려면 ffmpeg를 다시 컴파일해야한다고 한다.
> - 대충 아래 커맨드를 모두 때려넣었더니 된다.

## Nvida 관련 패키지 설치

### Nvidia Repostiroy Setup

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/3bf863cc.pub
sudo add-apt-repository 'deb https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/ /'
sudo apt install cuda -y
```

### Graphic Driver

```bash
sudo apt install nvidia-driver-525  nvidia-cuda-toolkit -y
```

> 내가 설치한 당시에는 525 버전이 최신이었다. 더 최신 버전이 apt 저장소에 있는지 찾아보자
{.is-info}

- nvidia 관련 설치가 끝났다면 우분투를 재부팅한다.
- WSL의 경우 cmd를 열고 `wsl --shutdown` 를 입력하면 WSL이 종료된다.

## Compile ffmpeg

### pre install

```bash
sudo apt install build-essential yasm cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev openssl libssl-dev libass-dev libfdk-aac-dev libmp3lame-dev libopus-dev libtheora-dev libvorbis-dev libvpx-dev libx264-dev libx265-dev -y
```

### nv-codec-headers

```bash
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git && cd nv-codec-headers && sudo make install
```

### ffmpeg

```bash
git clone https://git.ffmpeg.org/ffmpeg.git --depth=1 && cd ffmpeg
```

- ffmpeg Git 레포지터리 히스토리가 넘 커서 depth 1 만 가져오는게 편하다.

#### ffmpeg configuration

```bash
./configure \
--enable-nonfree \
--enable-cuda-nvcc \
--enable-libnpp \
--extra-cflags=-I/usr/local/cuda/include \
--extra-ldflags=-L/usr/local/cuda/lib64 \
--enable-openssl \
--enable-gpl \
--enable-libass \
--enable-libfdk-aac \
--enable-libfreetype \
--enable-libmp3lame \
--enable-libopus \
--enable-libtheora \
--enable-libvorbis \
--enable-libvpx \
--enable-libx264 \
--enable-libx265 \
--enable-nonfree
```

- cuda 옵션 + ffmpeg에서 일반적으로 많이 사용하는 옵션들이 포함됨

#### ffmpeg compile

```bash
sudo make install
```

- `make install`이 완료되면 ffmpeg 디렉토리 하위에서 ffmpeg 바이너리 파일이 컴파일되었다.
- (`PATH`에 등록된 ffmpeg 말고) 컴파일된 ffmpeg을 실행시키고 아래 커맨드로 테스트할 수 있다.

```bash
ffmpeg -y -hwaccel cuda -i input.file output.file
```


## 기타

- CPU(Ryzen 3700X)만 사용해서 ffmpeg를 실행했을 때보다 약 2배 정도 성능이 좋아졌다.
- `nvidia-smi` 커맨드를 사용하면 현재 그래픽 카드를 사용 현황을 확인할 수 있다.

```bash
$ nvidia-smi
Sun Feb 19 19:53:48 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.78.01    Driver Version: 528.49       CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:0A:00.0  On |                  N/A |
|  0%   43C    P2    51W / 374W |   2375MiB / 16376MiB |      5%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      5287      G   /ffmpeg                         N/A      |
+-----------------------------------------------------------------------------+
```

### References

- [How to install FFmpeg with NVIDIA GPU acceleration on Linux*www.cyberciti.biz*](https://www.cyberciti.biz/faq/how-to-install-ffmpeg-with-nvidia-gpu-acceleration-on-linux)
- [Enabling GPU acceleration on Ubuntu on WSL2 with the NVIDIA CUDA Platform*ubuntu.com/tutorials*](https://ubuntu.com/tutorials/enabling-gpu-acceleration-on-ubuntu-on-wsl2-with-the-nvidia-cuda-platform)
- [How to compile ffmpeg with https support*askubuntu.com*](https://askubuntu.com/a/956201)
- [[Ubuntu 20.04 LTS]Nvidia드라이버 설치하기*pstudio411.tistory.com*](https://pstudio411.tistory.com/entry/Ubuntu-2004-Nvidia%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)
{.links-list}