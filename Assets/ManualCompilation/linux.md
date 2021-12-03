# Manual compilation FFmpeg on Ubuntu 20.01

<br>

This guide is up to date with `FFmpeg` version `4.3.3` and `cuda` version `11.5`, for `ubuntu 20.01`, and writen in the winter 2021.

<br>

Those steps might change in the future - Nvidia Gpu Toolkit or flags to configure FFmpeg -.

<br>

# FFMPEG with NVIDIA GPU Computing Toolkit

<br>

Download Nvidia Cuda Toolkit from [here](https://developer.nvidia.com/cuda-downloads).

<br>

Download Nvidia GPU Computing Toolkit from [here](https://developer.nvidia.com/nvidia-video-codec-sdk).


<br>

Follow [this guide](https://docs.nvidia.com/video-technologies/video-codec-sdk/ffmpeg-with-nvidia-gpu/#compiling-for-linux) for the `nv headers` :

<br>

In the terminal:

<br>

```sh
mkdir /tmp/FFmpeg_install
FFMPEG_INSTALL=/tmp/FFmpeg_install
cd $FFMPEG_INSTALL
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git                     
cd nv-codec-headers
make -j$(nproc)
sudo make install
```

<br>

Install necessary libraries :

```sh
sudo apt-get update -qq && sudo apt-get -y install --assume-yes \
  autoconf \
  automake \
  build-essential \
  cmake \
  cmake-curses-gui \
  git-core \
  libass-dev \
  libfreetype6-dev \
  libsdl2-dev \
  libtool \
  libva-dev \
  libvdpau-dev \
  libvorbis-dev \
  libxcb1-dev \
  libxcb-shm0-dev \
  libxcb-xfixes0-dev \
  pkg-config \
  texinfo \
  wget \
  zlib1g-dev \
  libxml2-dev \
  yasm \
  libnuma-dev \
  libvpx-dev\
  libfdk-aac-dev \
  libmp3lame-dev \
  libx264-dev \
  libopus-dev \
  libavahi-client-dev \
  libavahi-common-dev
```

# x265 compilation

<br>

In the terminal :

<br>

```sh
FFMPEG_INSTALL=/tmp/FFmpeg_install
cd $FFMPEG_INSTALL
cd https://github.com/videolan/x265.git
cd x265
cmake -DCMAKE_INSTALL_PREFIX=$FFMPEG_INSTALL -DENABLE_SHARED=on ../../source
make -j$(nproc)
make install
```

# FFmpeg compilation

<br>


Download `FFmpeg 4.3.3` from [here](https://github.com/FFmpeg/FFmpeg/releases/tag/n4.3.3).

From the terminal :


```sh
FFMPEG_INSTALL=/tmp/FFmpeg_install
cd $FFMPEG_INSTALL
make clean
./configure \
--extra-cflags=-I/usr/local/cuda/include \
--extra-ldflags=-L/usr/local/cuda/lib64 \
--disable-static \
--enable-shared \
--prefix=/tmp/FFmpeg_install \
--extra-libs="-lpthread -lm" \
--pkg-config=pkg-config \
--disable-debug \
--enable-nonfree \
--enable-fontconfig \
--enable-libfdk-aac \
--enable-gnutls \
--enable-gpl \
--enable-libass \
--enable-libfreetype \
--enable-libmp3lame \
--enable-librtmp \
--enable-libtheora \
--enable-libvorbis \
--enable-libx264 \
--enable-libx265 \
--enable-libxvid \
--enable-libvpx \
--enable-pic \
--enable-postproc \
--enable-runtime-cpudetect \
--enable-swresample \
--enable-version3 \
--enable-zlib \
--disable-doc \
--enable-libmp3lame \
--enable-libopus \
--enable-libxml2 \
--enable-demuxer=dash \
--logfile=config.log \
--enable-cuda-nvcc \
--enable-libnpp

make -j$(nproc)
make install
```

<br>
