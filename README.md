# DECEASED: FFMPEG with NewTek NDI® support on MacOS

*NDI support in FFmpeg was removed in the FFMPEG source on the 20th of March 2019.

https://github.com/FFmpeg/FFmpeg/commit/4b32f8b3ebfa011fcc5991bcaa97c3f5b61b49ad

This is from a license dispute with Newtek ( https://trac.ffmpeg.org/ticket/7589 ) and NDI support has now been removed from FFmpeg.

I will take these projects offline.

Apologies for the inconvenience.

Regards,

Johan Els
*

# OLD README.md

Because of the NewTek NDI® SDK license agreement, we cannot distribute the SDK with FFmpeg directly. Here I've tried to make compiling it as simple as possible and then use this version to build a point2point NDI link over the Internet.

The theory is that if you can get NDI into and out of FFmpeg, FFmpeg has a documented point2point streaming documented on their site.

## Step 1 - Getting the SDK

Register and request the NewTek NDI® Software Developer Kit download link from https://www.newtek.com/ndi/sdk/#download-sdk and then download the MacOS version and install it. It dumps itself in the root of the drive, which I'm not happy about, but what can we do?

## Step 2 - Build environment

Install build tools using Brew. If you are not using brew or want to compile the dependencies as well, please see the page on the FFmpeg site for details.


```bash
brew install automake git nasm shtool texi2html theora wget \
    fdk-aac lame opus sdl x264 x265 xvid \
    libass libtool libvorbis libvpx
```

### Step 3 - Get the FFmpeg source

Clone FFmpeg repo from there GIT repo.

```bash
git clone http://source.ffmpeg.org/git/ffmpeg.git ffmpeg
cd ffmpeg
```

### Step 4 - Link into NewTek NDI® SDK

Symbolic links to resolve locations:
```bash
sudo ln -s /NewTek\ NDI\ SDK/ ndi
sudo ln -s /usr/local/lib/libndi.3.dylib /usr/local/lib/libndi.dylib
```

### Step 5 - Configure the FFmpeg build

Here I give two options. The first is with most of the FFmpeg options enabled. This results in a much bigger build, but also much better support.

#### Option 1 - FFmpeg large build

```bash
./configure  --prefix=/usr/local \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libndi_newtek \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-libxvid \
  --enable-nonfree \
  --extra-cflags="-I$PWD/ndi/include" \
  --extra-ldflags="-L$PWD/ndi/lib/x64" \
  --extra-libs="-lpthread -lm" \
  --samples=fate-suite/
```

#### Option 2 - FFmpeg simple build

```bash
./configure --enable-nonfree \
  --enable-gpl \
  --enable-libndi_newtek \
  --enable-libx264 \
  --enable-nonfree \
  --extra-cflags="-I$PWD/ndi/include" \
  --extra-ldflags="-L$PWD/ffmpeg/ndi/lib/x64"
```

### Step 6 - Let it build

```bash
make
```

### Step 7 - Quick test

This works on the same machine, from multiple terminals.

```bash
./ffmpeg -i ~/Downloads/input.mkv -f libndi_newtek -clock_video true -clock_audio true -pix_fmt uyvy422 "OUTPUT"

./ffmpeg -f libndi_newtek -find_sources 1 -i dummy

./ffplay -f libndi_newtek -i "MACBOOK.LOCAL (OUTPUT)"
```

## TX & RX samples
```bash
# TX & RX
./ffmpeg -i ~/Downloads/input.mkv -f mpegts - | ./ffplay -i -

# TX
./ffmpeg -re -i ~/Downloads/input.mkv -f mpegts "udp://192.168.33.159:1234?pkt_size=1316"
# RX
./ffmpeg -i udp://:1234 -f mpegts - | ./ffplay -

# TX
./ffmpeg -f libndi_newtek -i "MACBOOK (OUTPUT)" -f mpegts "udp://192.168.33.159:1234?pkt_size=1316"
# RX
./ffmpeg -i udp://:1234 -f libndi_newtek -clock_video true -clock_audio true -pix_fmt uyvy422 "OUTPUT"
```

## References:
* FFmpeg - https://www.ffmpeg.org
** Streaming Guide - https://trac.ffmpeg.org/wiki/StreamingGuide
** MacOS Compiling yourself -  https://trac.ffmpeg.org/wiki/CompilationGuide/macOS#CompilingFFmpegyourself
* NewTek NDI® SDK - https://www.newtek.com/ndi/sdk/
* NewTek NDI® port information - https://support.newtek.com/hc/en-us/articles/218109497-NDI-Video-Data-Flow
