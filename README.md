# 1.安装 Docker for Ubuntn 16.04

参考链接： https://docs.docker.com/engine/installation/linux/ubuntulinux/

##Setp1:

``` bash
sudo apt-get update

```

Setp2:

```   bash
   sudo apt-get installapt-transport-https ca-certificates
```

Setp3:

``` bash
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys58118E89F3A912897C070ADBF76221572C52609D
```

Setp4:

``` bash
    echo 'deb https://apt.dockerproject.org/repo ubuntu-xenial main' &gt; /etc/apt/sources.list.d/docker.list
```

PS: 其他版本Ubuntu，将语句中的<code>‘ ubuntu-xenial ’</code> 替换成对应的版本号
<table>
<thead>
<tr>
  <th align="center">Ubuntu Precise 12.04 (LTS)</th>
  <th align="center">Ubuntu Trusty 14.04 (LTS)</th>
  <th align="center">Ubuntu Wily 15.10</th>
  <th align="center">Ubuntu Xenial 16.04 (LTS)</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="center">ubuntu-precise</td>
  <td align="center">ubuntu-trusty</td>
  <td align="center">ubuntu-wily</td>
  <td align="center">ubuntu-xenial</td>
</tr>
</tbody>
</table>

Setp5:

```  bash
   apt-get update &amp;&amp; apt-get install -y -q docker-engine &amp;&amp; service docker start
```

其他linux版本，请参考 https://docs.docker.com/engine/installation/linux/

<ul>
<li>建立FFMPEG images</li>
</ul>

参考项目： https://github.com/cellofellow/ffmpeg

<ul>
<li>创建任意目录并进入目录</li>
</ul>

``` bash
    Mkdir xxxxx  &amp;&amp; cd xxxxx
```

<ul>
<li>创建Dockerfile</li>
</ul>

``` bash
      Vim Dockerfile
```

<ul>
<li>拷贝如下内容：</li>
</ul>

``` Docker
FROM ubuntu:16.04

MAINTAINER aeli li <aeli@qq.com>

# Set Locale

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

ENV FFMPEG_VERSION    3.0
  # monitor releases at https://github.com/FFmpeg/FFmpeg/releases
ENV YASM_VERSION      1.3.0
  # monitor releases at https://github.com/yasm/yasm/releases
ENV FDKAAC_VERSION    0.1.4
  # monitor releases at https://github.com/mstorsjo/fdk-aac/releases
#ENV  x264
  # this project does not use release versions at this time
  # monitor project at http://git.videolan.org/?p=x264.git;a=shortlog
#ENV  LAME_VERSION      3.99.5
#ENV  FAAC_VERSION      1.28
#ENV  XVID_VERSION      1.3.3
#ENV  MPLAYER_VERSION   1.1.1
ENV BLACKMAGIC_SDK_VERSION  10.5.4
  # monitor my own releases at https://github.com/nachochip/Blackmagic-SDK/releases
  # the origin of the drivers comes from https://www.blackmagicdesign.com/support/family/capture-and-playback
  # I roll them into github to track it better, and condense to only linux-drivers
ENV SRC               /usr/local
ENV BUILD             ${SRC}/ffmpeg_build
ENV LD_LIBRARY_PATH   ${SRC}/lib
ENV PKG_CONFIG_PATH   ${SRC}/lib/pkgconfig

# Install. ubuntu

RUN  sed -i 's/# (.*multiverse$)/1/g' /etc/apt/sources.list  \
  && apt-get update \
  && apt-get -y upgrade \
  && apt-get -q -y install build-essential \
  && apt-get -q -y install software-properties-common \
  && apt-get -q -y install byobu curl git htop man unzip vim wget \
  && apt-get -q -y clean \
  && rm -rf /var/lib/apt/lists/*

# Install FFMPEG need lib.
RUN echo 'deb http://archive.ubuntu.com/ubuntu precise universe multiverse' >> /etc/apt/sources.list \
  && apt-get update \
  && apt-get -y -q install autoconf automake build-essential libass-dev libfreetype6-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libmp3lame-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev \
  && apt-get clean

# Run build script

# ADD build.sh /build.sh
# RUN ["/bin/bash", "/build.sh"]

# YASM
RUN DIR=$(mktemp -d) && cd ${DIR} \
              && curl -Os http://www.tortall.net/projects/yasm/releases/yasm-${YASM_VERSION}.tar.gz \
              && tar xzvf yasm-${YASM_VERSION}.tar.gz \ 
              && cd yasm-${YASM_VERSION} \
              && ./configure --prefix="${BUILD}" --bindir="${SRC}/bin" \
              && make \
              && make install \
              && make distclean \
              && rm -rf ${DIR}

# x264
RUN DIR=$(mktemp -d) && cd ${DIR} \ 
              && wget http://download.videolan.org/pub/x264/snapshots/last_x264.tar.bz2 \
              && tar xjvf last_x264.tar.bz2 \
              && cd x264-snapshot* \
              && PATH="${SRC}/bin:$PATH" ./configure --prefix="${SRC}" --bindir="${SRC}/bin" --enable-static --disable-opencl \
              && PATH="${SRC}/bin:$PATH" make \
              && make install \
              && make clean \
              && rm -rf ${DIR}
             
# x265
RUN DIR=$(mktemp -d) && cd ${DIR} \ 
        && apt-get install -y -q cmake mercurial \
        && hg clone https://bitbucket.org/multicoreware/x265 \
        && cd x265/build/linux \
        && PATH="${SRC}/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="${SRC}" -DENABLE_SHARED:bool=off ../../source \
        && make \
        && make install \
        && make clean \
        && rm -rf ${DIR}

# LAME
#RUN DIR=$(mktemp -d) && cd ${DIR} \
#              && curl -L -Os http://downloads.sourceforge.net/project/lame/lame/${LAME_VERSION%.*}/lame-${LAME_VERSION}.tar.gz  && \
#              tar xzvf lame-${LAME_VERSION}.tar.gz  && \
#              cd lame-${LAME_VERSION} && \
#              ./configure --prefix="${SRC}" --bindir="${SRC}/bin" --disable-shared --enable-nasm && \
#              make && \
#              make install && \
#              make distclean && \ 
#              rm -rf ${DIR}

# FAAC
  # This combines faac + http://stackoverflow.com/a/4320377
#RUN DIR=$(mktemp -d) && cd ${DIR} && \
#              curl -L -Os http://downloads.sourceforge.net/faac/faac-${FAAC_VERSION}.tar.gz  && \
#              tar xzvf faac-${FAAC_VERSION}.tar.gz  && \
#              cd faac-${FAAC_VERSION} && \
#              sed -i '126d' common/mp4v2/mpeg4ip.h && \
#              ./bootstrap && \
#              ./configure --prefix="${SRC}" --bindir="${SRC}/bin" && \
#              make && \
#              make install && \
#              rm -rf ${DIR}

# XVID
#RUN DIR=$(mktemp -d) && cd ${DIR} && \
#              curl -L -Os  http://downloads.xvid.org/downloads/xvidcore-${XVID_VERSION}.tar.gz  && \
#              tar xzvf xvidcore-${XVID_VERSION}.tar.gz && \
#              cd xvidcore/build/generic && \
#              ./configure --prefix="${SRC}" --bindir="${SRC}/bin" && \
#              make && \
#              make install && \
#              rm -rf ${DIR}

# FDK_AAC
RUN DIR=$(mktemp -d) && cd ${DIR} && \
              curl -s https://codeload.github.com/mstorsjo/fdk-aac/tar.gz/v${FDKAAC_VERSION} | tar zxvf - && \
              cd fdk-aac-${FDKAAC_VERSION} && \
              autoreconf -fiv && \
              ./configure --prefix="${SRC}" --disable-shared && \
              make && \
              make install && \
              make distclean && \
              rm -rf ${DIR}

# Blackmagic SDK
RUN cd /usr/src/ && \
        curl -s https://codeload.github.com/nachochip/Blackmagic-SDK/tar.gz/${BLACKMAGIC_SDK_VERSION} | tar xzvf -

# FFMPEG
  # I removed these flags from configure:  --enable-libfaac --enable-libmp3lame  --enable-libxvid
RUN DIR=$(mktemp -d) && cd ${DIR} && \
              curl -Os http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz && \
              tar xzvf ffmpeg-${FFMPEG_VERSION}.tar.gz && \
              cd ffmpeg-${FFMPEG_VERSION} && \
              ./configure --prefix="${SRC}" --extra-cflags="-I${SRC}/include" --extra-ldflags="-L${SRC}/lib" --bindir="${SRC}/bin" \
              --pkg-config-flags="--static" \
              --extra-libs=-ldl --enable-version3 --enable-libx264 --enable-libx265 --enable-gpl \
              --enable-postproc --enable-nonfree --enable-avresample --enable-libfdk_aac --disable-debug  --enable-small \
              --enable-decklink --extra-cflags=-I/usr/src/Blackmagic-SDK-${BLACKMAGIC_SDK_VERSION}/Linux/include/ \
              --extra-ldflags=-L/usr/src/Blackmagic-SDK-${BLACKMAGIC_SDK_VERSION}/Linux/include/ && \
              make && \
              make install && \
              make distclean && \
              hash -r && \
              rm -rf ${DIR}

RUN echo "/usr/local/lib" > /etc/ld.so.conf.d/libc.conf

CMD           ["--help"]
ENTRYPOINT    ["ffmpeg"]
```

保存并退出vim或其他编辑器。

执行命令生成images：

``` bash
Docker build -t='image镜像标签名' .
```

文件转码输出（点播HSL分段）：

``` bash
//docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -c:v libx264 -c:a aac -f hls -&gt; /本机输出目录/文件名.m3u8

sudo docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -c:v libx264 -c:a aac -map 0 -flags -global_header -f ssegment -segment_time 10 -segment_format mpegts -segment_list /容器挂载目录/媒体输出播放列表.m3u8 /容器挂载目录/媒体输出文件名%03d.ts

// %03d代表有3位的数字顺序符，会生成001、002这样的文件名，如果文件过大或过小，可以自定义数字位数。
```