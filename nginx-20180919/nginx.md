**1.nginx简介：**

    Nginx是一个高性能的 Web 和反向代理服务器, 它具有有很多非常优越的特性:
        1.作为 Web 服务器：相比 Apache，Nginx 使用更少的资源，
            -1.支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。
            -2.能够支持高达 50,000 个并发连接数的响应，感谢 Nginx 为我们选择了 epoll and kqueue 作为开发模型.
        2.作为负载均衡服务器：Nginx 既可以在
            -1.内部直接支持 Rails 和 PHP，也可以支持作为 HTTP代理服务器 对外进行服务。
            -2.Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。
        3.作为邮件代理服务器: Nginx 同时也是一个非常优秀的邮件代理服务器(最早开发这个产品的目的之一也是作为邮件代理服务器)，Last.fm 描述了成功并且美妙的使用经验。
            -1.Nginx 安装非常的简单，配置文件 非常简洁(还能够支持perl语法)
            -2.Bugs非常少的服务器: Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。
            -3.还能够在 不间断服务的情况下进行软件版本的升级
            
            
**2.nginx安装：**

    yum install gcc-c++
    yum install -y pcre pcre-devel
    yum install -y zlib zlib-devel
    yum install -y openssl openssl-devel
    wget http://nginx.org/download/nginx-1.8.1.tar.gz
    tar -zxvf /usr/local/nginx-1.8.0.tar.gz
    cd /usr/local/nginx-1.8.0
    ./configure --prefix=/usr/local/nginx
    make
    make install
    
    ++目录介绍：+++++++++++++++++++++++++++++++++++++++++++++
    cd /usr/local/nginx
        conf    配置文件
        html    网页文件
        logs    日志文件
        sbin    主要二进制程序
    ++启动nginx：+++++++++++++++++++++++++++++++++++++++++++++
    ./sbin/nginx
        ——————————
        1.若80端口被占用：
            -nginx:[emerg] bind() to 0.0.0.:80 failed (98: Address already in use)
            -解决：
                -关闭占用的80端口：
                    1.linux自带的apache自动启动：service httpd stop
                    2.自己编译安装的：#path/to/apacht/bin/apachctl stop
                    3.没有被占用，仍报错：nginx可能同时监听IPV4和6的80端口导致：
                        server {
                            listen:80;
                            listen [::]:80;
                        }
                        修改为：
                        server {
                            listen:80;
                            listen [::]:80 ipv6only=on;
                        }
                        或
                        server {
                            listen [::]:80;
                        }
    ++命令：+++++++++++++++++++++++++++++++++++++++++++++
    1.查看nginx是否启动：可获取pid
        ps aux|grep nginx
        ps -ef|grep nginx
    2.直接查看文件获取pid：
        cat /usr/local/nginx/logs/nginx.pid 
    3.测试配置是否正确：
        ./sbin/nginx -t
    4.平滑的重启。配置重载：
        ./sbin/nginx -s reload 		
            一个master进程：
            三个slave子进程：当reload开始时，等待子进程执行完毕后，立即杀掉，并重新开始另一个进程（此为最新配置的）
            逐个进行替换，直到所有旧进程替换完毕
    5.快速停止：不等，直接停止：
        ./sbin/nginx -s stop
    6.平滑停止：--等待当前进程完成之后停止，即访问结束后停止
        ./sbin/nginx -s quit
    7.重新打开访问日志记录：--防止nginx文件日志过大,日志切分
        mv /usr/local/nginx/logs/access.log /usr/local/nginx/logs/access-20190920.log 
        touch /usr/local/nginx/logs/access.log 
        ./sbin/nginx -s reopen
        
**2.nginx配置：**

    cd /usr/local/nginx/conf
    1.vim nginx.conf    --同目录下查看
    
**3.日志管理：**
    
    1.每个server可配置不同日志：--同目录下查看
 
**3.url重写：**   

    1.rewrite
    2.try_files
    
**4.反向代理;防止丢失真实用户ip---在logs中可查看**

     location ~ \.(jpeg|jpg|png|gif)$ {
                proxy_pass http://192.168.1.230:9200;
                #防止丢失真实用户ip
                proxy_set_header X-Forwarded-For $remote_addr
     }
**5.负载均衡：**  

    1.配置服务器组（上游服务器）：--upstream
    2.配置location

**6.负载均衡策略：**

    1.session丢失问题：--
    2.默认负载均衡算法：
        设置计数器鲁迅请求n台服务器
      --可安装3方模块，利用不同参数做均衡
        1.基于cookie值区别用户做负载均衡-nginx sticky模块
        2.基于uri利用一致性哈希算法
        3.利用ip做负载
**7.配置基于用户的访问密码：**

    yum -y install httpd 
    htpasswd -c ./auth_conf  root;
        输入两次密码，当前路径下生成文件：auth_conf
        -c：默认是使用md5加密， 
        ./auth_conf 是指定路径和文件 ， 
        root是用户名
     location / {
                auth_basic "user and pwd";
                auth_basic_user_file /usr/local/nginx/auth_conf;
                proxy_pass http://192.168.1.230:8080;
                #防止丢失真实用户ip
                proxy_set_header X-Forwarded-For $remote_addr;
                }

    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    