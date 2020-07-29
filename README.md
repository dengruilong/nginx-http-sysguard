##Description

This module can be used to protect your server in case system load or memory use goes too high.<br/>
It comes from [Tengine](http://tengine.taobao.org), an Nginx distribution with quite a few advanced features.

###Examples:

    server {
        sysguard on;

        sysguard_load load=10.5 action=/loadlimit;
        sysguard_mem swapratio=20% action=/swaplimit;

        location /loadlimit {
            return 503;
        }

        location /swaplimit {
            return 503;
        }
    }

##Installation

    $ wget http://www.nginx.org/download/nginx-1.2.5.tar.gz
    $ tar xzvf nginx-1.2.5.tar.gz
    
    $ cd nginx-1.2.5
    $ git clone https://github.com/taobao/nginx-http-sysguard.git
    $ patch -p1 < nginx-http-sysguard/nginx_sysguard_1.2.5.patch
    
    $ ./configure --add-module=./nginx-http-sysguard
    # make && make install

##Directives

**Syntax**: ***sysguard*** [on | off]  
**Default**: sysguard off  
**Context**: http, server, location  

Turn on or off this module.
<br/>

**Syntax**: ***sysguard_load*** load=number [action=/url]  
**Default**: none  
**Context**: http, server, location  

Specify the load threshold. When the system load exceeds this threshold, all subsequent requests will be redirected to the URL specified by the 'action' parameter. Nginx will return 503 if there's no 'action' URL defined.
<br/>

**Syntax**: ***sysguard_mem*** swapratio=ratio% [action=/url]  
**Default**: none  
**Context**: http, server, location  

Specify the used swap memory threshold. When the swap memory use ratio exceeds this threshold, all subsequent requests will be redirected to the URL specified by the 'action' parameter. Nginx will return 503 if there's no 'action' URL.
<br/>

**Syntax**: ***sysguard_interval*** time  
**Default**: sysguard_interval 1s  
**Context**: http, server, location  

Specify the time interval to update your system information. The default value is one second, which means Nginx updates the server status once a second.
<br/>

**Syntax**: ***sysguard_log_level*** [info | notice | warn | error]  
**Default**: sysguard_log_level error  
**Context**: http, server, location  
Specify the log level of sysguard.
<br/>


### 下载sysguard

$wget https://github.com/alibaba/nginx-http-sysguard/archive/master.zip -O /tmp/nginx-http-sysguard-master.zip

$unzip nginx-http-sysguard-master.zip

### 下载openresty

$cd /tmp

$wget https://openresty.org/download/ngx_openresty-1.9.3.1.tar.gz

### 编译安装，先要打一个patch 到 openresty 的nginx core中, 然后编译安装

$unzip nginx-http-sysguard-master.zip

$tar zxvf ngx_openresty-1.9.3.1.tar.gz

$cd ngx_openresty-1.9.3.1/bundle/nginx-1.9.3/

$patch -p1 < ../nginx-http-sysguard-master/nginx_sysguard_1.3.9.patch

$cd ../../

$./configure --with-luajit --with-http_stub_status_module --add-module=/tmp/nginx-http-sysguard-master/

$ gmake && gmake install

$/usr/local/openresty/nginx/sbin/nginx -V 查看版本，sysguard是否安装上

### 配置
在其中的一个server配置中，加入如下进行配置，然后重载配置nginx配置文件

Server {

listen      80;

server_name  localhost;

root /mnt/htdocs;

error_page  500 502 503 504  /50x.html;

location = /50x.html {

             root  html;

}

sysguard on;

sysguard_load load=0.01 action=/50x.html;

sysguard_mem swapratio=20% action=/50x.html;

}

### 运行测试
使用命令$uptime 查看服务器负载

如果不高使用 $ab -c 100 -n 10000 http://localhost 压测，增加负载

在$curl http://127.0.0.1 访问将返回50x.html 证明保护生效了
