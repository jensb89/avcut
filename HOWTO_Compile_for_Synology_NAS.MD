# TUTORIAL
## Compile avcut for Synology NAS with a cross compiler (Synology Toolchain)

### Step 1: Donwload DSM Toolchain + make src + build folders
Go to [https://sourceforge.net/projects/dsgpl/files/?source=navbar](https://sourceforge.net/projects/dsgpl/files/?source=navbar) and download the toolchain for your system (DSM 6.0, Marvel Armada 38x for the DS 116) 

Unpack it to \usr\local as pointed out by Synology ([here](https://developer.synology.com/developer-guide/compile_applications/download_dsm_tool_chain.html))

Open a terminal a some new folder and for simplicity create two new folders here:
```shell
mkdir src
mkdir build
cd src
```

Export the Path:
```shell
export PATH=$PATH:/usr/local/arm-unknown-linux-gnueabi/bin
export CCPREFIX="/usr/local/arm-unknown-linux-gnueabi/bin/arm-unknown-linux-gnueabi-"
```

### Step 2: Compile x264
You should be in the src folder now. Then: 
```shell
git clone git://git.videolan.org/x264
cd x264
./configure --host=arm-unknown-linux-gnueabi --enable-static --cross-prefix=${CCPREFIX} --prefix=../../build 
make
make install
```

### Step 3: Compile FFMPEG with x264
I used 2.8.6 for compiling, newer versions might not work with avcut...

```shell
cd ..
git clone git://source.ffmpeg.org/ffmpeg.git
cd ffmpeg
git checkout n2.8.6
pkg_config=$(which pkg-config) PKG_CONFIG_PATH=../../build/lib/pkgconfig ./configure --enable-cross-compile --cross-prefix=${CCPREFIX} --arch=armel --target-os=linux --disable-programs --disable-hwaccels --disable-indevs --disable-outdevs --disable-devices --enable-static --disable-zlib --disable-xlib --disable-sdl --disable-libxcb --disable-bzlib --disable-libxcb-shm --disable-libxcb-xfixes --disable-libxcb-shape --disable-iconv --enable-gpl --enable-libx264 --disable-encoder=tiff --disable-decoder=tiff --prefix=../../build --extra-cflags="-I../../build/include" --extra-ldflags="-L../../build/lib" --extra-libs=-ldl
make
make install
```

### Step 4: Compile AVCUT without makefile
I used a fork with a patch from monarc99, which also is a fork from anyc (originator). 
The direct version may also work...

In order for avcut to compile I had to rename the time.h file in build/include/libavutils. See [this](http://stackoverflow.com/questions/14947691/c-system-file-bits-stat-h-suddenly-breaks-with-error-field-st-atim-has-inc) for more info on the error.
```shell
mv ../../build/include/libavutil/time.h ../../build/include/libavutil/time_.h
```

Code to compile avcut (I did not use the makefile):
```shell
export LDFLAGS="-L../../build/lib"
arm-unknown-linux-gnueabi-gcc -Wall -DAVCUT_VERSION=\"0.2\" -I../../build/include/libavcodec -I../../build/include/libavdevice -I../../build/include/libavfilter -I../../build/include/libavformat -I../../build/include/libavutil -I../../build/include/libpostproc -I../../build/include/libwresample -I../../build/include/libswscale -I../../build/include -L../../build/lib  avcut.c  ../../build/lib/libavformat.a ../../build/lib/libavcodec.a ../../build/lib/libavutil.a ../../build/lib/libswresample.a ../../build/lib/libx264.a -lpthread -lm -ldl -static -o avcut
```

After this the avcut binary should be in the src/avcut folder!

### Alternative Step 4: Compile with makefile
For newer avcut version one can also try with the makefile:

```shell
export CROSS_COMPILE=arm-unknown-linux-gnueabi-
export CPPFLAGS="-I../../build/include/libavcodec -I../../build/include/libavdevice -I../../build/include/libavfilter -I../../build/include/libavformat -I../../build/include/libavutil -I../../build/include/libpostproc -I../../build/include/libwresample -I../../build/include/libswscale -I../../build/include"
export LDFLAGS="-L../../build/lib"
make static
```

or

```shell
LDLIBS="../../build/lib/libavformat.a ../../build/lib/libavcodec.a ../../build/lib/libavutil.a ../../build/lib/libswresample.a ../../build/lib/libx264.a -lpthread -lm -static" make -I../../build/lib -e
```

For me at least -ldl was missing then...
