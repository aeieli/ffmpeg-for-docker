

# 1.安装 Docker for Ubuntn 16.04

如果非Ubuntn系统，请参考其他版本的Docker安装教程

参考链接： https://docs.docker.com/engine/installation/linux/ubuntulinux/

## Setp1:

``` bash
sudo apt-get update

```

## Setp2:

```   bash
sudo apt-get installapt-transport-https ca-certificates
```

## Setp3:

``` bash
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys58118E89F3A912897C070ADBF76221572C52609D
```

## Setp4:

``` bash
echo 'deb https://apt.dockerproject.org/repo ubuntu-xenial main' &gt; /etc/apt/sources.list.d/docker.list
```

PS: 其他版本Ubuntu，将语句中的<code>‘ ubuntu-xenial ’</code> 替换成对应的版本号

|    |   |   |    |
| :----  | :----  |:----  |:----  |

| Ubuntu Precise 12.04 (LTS)| Ubuntu Trusty 14.04 (LTS)| Ubuntu Wily 15.10 | Ubuntu Xenial 16.04 (LTS) |
|ubuntu-precise |ubuntu-trusty | ubuntu-wily | ubuntu-xenial |

## Setp5:

```  bash
apt-get update &amp;&amp; apt-get install -y -q docker-engine &amp;&amp; service docker start

```

其他linux版本，请参考 https://docs.docker.com/engine/installation/linux/

# 2. 建立FFMPEG images 

参考项目： https://github.com/cellofellow/ffmpeg

## 创建任意目录并进入目录

``` bash
Mkdir xxxxx && cd xxxxx
```

## 下载 Dockerfile


``` bash

wget https://raw.githubusercontent.com/aeieli/ffmpeg-for-docker/master/Dockerfile

```

## 执行命令生成images：

``` bash
Docker build -t='image镜像标签名' .
// 请特别注意参数最后的.符号
```

# 文件转码输出（点播HSL分段）：

``` bash

sudo docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -c:v libx264 -c:a aac -map 0 -flags -global_header -f ssegment -segment_time 10 -segment_format mpegts -segment_list /容器挂载目录/媒体输出播放列表.m3u8 /容器挂载目录/媒体输出文件名%03d.ts

// %03d代表有3位的数字顺序符，会生成001、002这样的文件名，如果文件过大或过小，可以自定义数字位数。


```
可选参数： -i 输入文件后，可以设定以下参数来确定文件输出码率、格式等
`-b 128k  (设置比特率，缺省200kb/s）`
```
sudo docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -b 128k -c:v libx264 -c:a aac -map 0 -flags -global_header -f ssegment -segment_time 10 -segment_format mpegts -segment_list /容器挂载目录/媒体输出播放列表.m3u8 /容器挂载目录/媒体输出文件名%03d.ts

```

-r fps 设置帧频 缺省25

-aspect aspect 设置横纵比 4:3 16:9 或 1.3333 1.7777

-s size 设置帧大小 格式为WXH 缺省160X128.下面的简写也可以直接使用：Sqcif 128X96 qcif 176X144 cif 252X288 4cif 704X576 

其他可以参考文档： https://www.ffmpeg.org/ffmpeg-formats.html#hls-1

http://www.cnblogs.com/vicowong/archive/2011/03/08/1977088.html

# License

MIT License