# Nginx HLS Server配置

## 1.软件编译
从下面的网址分别下载nginx和nginx-rtmp-module：
http://nginx.org/en/download.html
https://github.com/arut/nginx-rtmp-module

Nginx的安装和配置过程省略

Build Hls 支持模块：

```
\\cd to NGINX source directory & run this:

./configure --add-module=/path/to/nginx-rtmp-module
make
make install

```

如果是支持SSL模式的1.3.14~1.50版本的Nginx，使用下面的语句配置
```
./configure --add-module=/path/to/nginx-rtmp-module --with-http_ssl_module
make
make install

```

## 2. Nginx 参数配置

编辑nginx配置文件nginx.conf，增加以下内容：
``` html

rtmp {  
  
    server {  
  
        listen 1935;  
  
        chunk_size 4000;  
        
        #HLS  
  
        # For HLS to work please create a directory in tmpfs (/tmp/app here)  
        # for the fragments. The directory contents is served via HTTP (see  
        # http{} section in config)  
        #  
        # Incoming stream must be in H264/AAC. For iPhones use baseline H264  
        # profile (see ffmpeg example).  
        # This example creates RTMP stream from movie ready for HLS:  
        #  
        # ffmpeg -loglevel verbose -re -i movie.avi  -vcodec libx264   
        #    -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1   
        #    -f flv rtmp://localhost:1935/hls/movie  
        #  
        # If you need to transcode live stream use 'exec' feature.  
        #  
        application hls {  
            live on;  
            hls on;  
            hls_path /usr/local/nginx/html/hls;  
            hls_fragment 5s;  
        }  
    }  
}  

```

同时，在nginx.conf的http部分中，新增以下server:
``` html
	server {  
  
	        listen  8080;  
	        location /hls {  
	            # Serve HLS fragments  
	            types {  
	                application/vnd.apple.mpegurl m3u8;  
	                video/mp2t ts;  
	            }  
	            root html;  
	            expires -1;  
	        }  
	    }  
```

然后运行如下命令检查nginx.conf是否有语法错误
``` bash
service nginx configtest
```
重新加载配置文件
``` bash

service nginx reload
```
运行下面的命令查看nginx状态
``` bash
service nginx status
``` 
然后查看端口
``` bash
netstat -nlp
```
注意，这里reload并不能开启这几个服务，需要使用
``` bash
service nginx restart
```
来将rtmp服务和hls服务的端口1935和8080都打开

参考来源：

https://github.com/arut/nginx-rtmp-module/wiki/Directives

http://blog.csdn.net/tao_627/article/details/22271559


## 播放：

网页使用下面的地址，即可播放HLS文件

``` html
<video src="http://localhost/m3u8/index.m3u8"></video>

```