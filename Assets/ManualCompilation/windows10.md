# Manula compilation FFmpeg on Windows 10.

<br>

This guide is up to date with `FFmpeg` version `4.3.3` and `cuda` version `11.5`, for `Windows 10`, and writen in the winter 2021.

<br>

Those steps might change in the future - Nvidia Gpu Toolkit or flags to configure FFmpeg -.

# Install msys2 and mingw64


<br>

Download msys2 from [here](https://www.msys2.org/).

<br>

Follow the first 7 steps on the msys2 download page to set up msys2 and update the packages.

<br>

Make sure to install msys2 at `C:\dev\msys64` as this is the path referred to to launch it.

<br>

# x264

<br>

Open `x64 Native Tools Command Prompt for VS 2017`

```sh
cd C:\dev\msys64
msys2_shell.cmd -mingw64 -use-full-path
```

<br>

In `mingw64` :

```sh
# make sure path includes the x64 folder of MSVC correctly :
echo $PATH
# look for something like this :
/c/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/HostX64/x64
cd /
mkdir FFmpegInstall && cd FFmpegInstall && mkdir ffmpeg_build
git clone https://github.com/mirror/x264
cd x264
#configure
./configure --prefix="/FFmpegInstall/ffmpeg_build" \
  --enable-shared \
  --enable-static \
  --disable-win32thread \
  --enable-lto \
  --enable-debug \
  --enable-gprof \
  --enable-strip \
  --enable-pic

make 
make install

```

<br>
<br>


# x265

<br>

Download x265 source code from [here](https://bitbucket.org/multicoreware/x265_git/downloads/?tab=tags), I've used 3.5.

<br>

Unzip it to `C:\dev\msys64\FFmpegInstall` .

<br>

Open `make-solutions.bat` inside `build/vc15-x86_64` and change its content to :

<br>

```sh
cmake -G "Visual Studio 16 2019" -T "v142" -A x64 -DCMAKE_SYSTEM_VERSION=10.0 "..\..\source"
```

<br>


Open `x64 Native Tools Command Prompt for VS 2017` :

<br>

```sh
cd C:\dev\msys64\FFmpegInstall\x265\build\vc15-x86_64
.\make-solution.bat
```

<br>

Then open the `x265.sln` `Microsoft Visual studio 2019 solution` in `C:\dev\msys64\FFmpegInstall\x265\build\vc15-x86_64`

<br>

Compile it in Microsoft Visual Studio.

<br>

Copy the content of the `release` folder into the `lib` folder : 
`C:\dev\msys64\FFmpegInstall\x265\build\vc15-x86_64\Release` to `C:\dev\msys64\FFmpegInstall\ffmpeg\ffmpeg_build\lib`
Execept the `.exe` file `x265.exe` which goes to `C:\dev\msys64\FFmpegInstall\ffmpeg\ffmpeg_build\bin` .


<br>
<br>

# FFMPEG with NVIDIA GPU Computing Toolkit

<br>

Download Nvidia Cuda Toolkit from [here](https://developer.nvidia.com/cuda-downloads).

<br>

Download Nvidia GPU Computing Toolkit from [here](https://developer.nvidia.com/nvidia-video-codec-sdk).

<br>

Follow [this guide](https://docs.nvidia.com/video-technologies/video-codec-sdk/ffmpeg-with-nvidia-gpu/#compiling-for-windows) for the `nv headers` :

<br>

Open `x64 Native Tools Command Prompt for VS 2017`

```sh
cd C:\dev\msys64
msys2_shell.cmd -mingw64 -use-full-path
```

<br>

In `mingw64` :

```sh
cd /FFmpegInstall/
mkdir nv_sdk && mkdir nv_sdk/include && mkdir nv_sdk/lib
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers/
make 
make install
# dependencies
pacman -S diffutils make pkg-config yasm
# move all link
mv /usr/bin/link.exe /usr/bin/link.exe.bak
mv /bin/link.exe /bin/link.exe.bak
#then 
which link
#gives me
/c/Program Files (x86)/Microsoft Visual Studio/2019/Enterprise/VC/Tools/MSVC/14.29.30037/bin/Hostx64/x64/link
```

<br>

Copy content of the folders:
`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.5\include` to `C:\dev\msys64\FFmpegInstall\ffmpeg\nv_sdk\include`
and
`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.5\lib\x64` to `C:\dev\msys64\FFmpegInstall\ffmpeg\nv_sdk\lib`

<br>

<br>

## Get your graphic card architecture

<br>

Check that the installation of cuda has been successful, and check the capabilities of your graphic card (as detailed [here](https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/#verify-installation) ):

<br>

Open the `x64 Native Tools Command Prompt for VS 2017`:

<br>

```sh

C:\ProgramData\NVIDIA Corporation\CUDA Samples\v11.5\bin\win64\Release

```

This will give you the capabilities of the GPU code Generation (`6.0` will correspond to `compute_60` or below `compute_50` etc...).

<br>

Keep your GPU code Generation and add it as a flag to configuration comand below, such as :

<br>

for me: I had a graphic card with `5.2` so the code is `52`; the flag becomes:

<br>

```sh
--nvccflags="-gencode arch=compute_52,code=sm_52 -O2"
```


<br>

Download `FFmpeg 4.3.3` from [here](https://github.com/FFmpeg/FFmpeg/releases/tag/n4.3.3).

<br>

Unzip it into `C:\dev\msys64\FFmpegInstall` .

<br>

Open the `x64 Native Tools Command Prompt for VS 2017`:

<br>

```sh
cd C:\dev\msys64
msys2_shell.cmd -mingw64 -use-full-path
```

In the mingw64 shell :

<br>

```sh
cd /FFmpegInstall/FFmpeg-n4.3.3
make clean
./configure \
--toolchain=msvc \
--prefix="/FFmpegInstall/ffmpeg_build" \
--target-os=win64 \
--arch=x86_64 \
--extra-ldflags=-libpath:"/FFmpegInstall/nv_sdk/lib" \
--extra-ldflags=-libpath:"/FFmpegInstall/ffmpeg_build/lib" \
--extra-cflags=-I"/FFmpegInstall/nv_sdk/include" \
--extra-cflags=-I"/FFmpegInstall/ffmpeg_build/include" \
--nvccflags="-gencode arch=compute_52,code=sm_52 -O2" \
--toolchain=msvc \
--enable-cuda-nvcc \
--enable-libnpp \
--enable-shared \
--enable-avresample \
--pkg-config=pkg-config \
--disable-debug \
--enable-dxva2 \
--enable-d3d11va \
--enable-nonfree \
--enable-gpl \
--enable-libx264 \
--enable-version3 \
--disable-doc \
--logfile=config.log
make -j$(nproc)
make install
```

<br>