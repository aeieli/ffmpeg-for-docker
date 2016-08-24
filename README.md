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
<li>下载 Dockerfile</li>

</ul>

``` bash
  wget https://raw.githubusercontent.com/aeieli/ffmpeg-for-docker/master/Dockerfile
```

执行命令生成images：

``` bash
Docker build -t='image镜像标签名' .
// 请特别注意参数最后的.符号
```

文件转码输出（点播HSL分段）：

``` bash
//docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -c:v libx264 -c:a aac -f hls -&gt; /本机输出目录/文件名.m3u8

sudo docker run --rm -v /本机输入文件目录:/容器挂载目录 image镜像标签名 -i /容器挂载目录/媒体文件名 -c:v libx264 -c:a aac -map 0 -flags -global_header -f ssegment -segment_time 10 -segment_format mpegts -segment_list /容器挂载目录/媒体输出播放列表.m3u8 /容器挂载目录/媒体输出文件名%03d.ts

// %03d代表有3位的数字顺序符，会生成001、002这样的文件名，如果文件过大或过小，可以自定义数字位数。
```