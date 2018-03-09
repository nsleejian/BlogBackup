# Larveal 项目部署

#### 前言
 如何把一个已经做好的项目部署到服务器上呢？本文将为你解答，如有不足欢迎指正，不吝赐教。

####0 大纲
1. 准备一台服务器
2. 安装 Nginx
3. 安装 MySQL
4. 安装 PHP 
5. 修改 PHP 相关配置
6. 修改 Nginx 配置
7. 修改项目相关配置

####1 准备一台服务器
部署之前你需要一台服务器，阿里云，腾讯云，Digitalocean，Linode等云服务器都可以。国内的服务器速度相对较快，但是需要备案；国外的速度相对慢一下，无需备案。建议如果长期使用，还是用国内的。本文内容以阿里云服务器为例，操作系统为 Ubuntu 16.04 LTS。Ubuntu 、CentOS 各有所爱，无需多讲。

#### 2 安装 Nginx
想要部署，首先你需要一台 Web 服务器，以 Nginx 为例。
安装 Nginx :

```
sudo apt -y install nginx
```
等待命令执行完成之后，在浏览器中输入你服务器的 ip 地址应该就可以看到 Nginx 的默认页面了。

####3 安装 MySQL

服务中需要数据的存储，以 MySQL 为例。
安装 MySQL :

```
sudo apt install mysql-server
```
安装过程中会要求你输入密码，按要求输入即可。

####4 安装 PHP
安装 PHP 相对会复杂一些，因为还要安装想过模块。

* 4.1 安装 PHP 7.1

```
sudo apt -y install php7.1
```
* 4.2 安装 MySQL 通信模块 , 用于和 MySQL 的通信。

```
sudo apt -y install php7.1-mysql
```
* 4.3 安装 fpm 模块，用于 PHP 解析。

```
sudo apt install php7.1-fpm
```
* 4.3 安装 zip unzip php7.0-zip 模块。

```
sudo apt install zip unzip php7.0-zip
```
* 4.4 安装 php7.1-curl 模块。

```
sudo apt install  php7.1-curl
```
* 4.5 安装其他模块

```
sudo apt install php7.1-json php7.1-mbstring php7.1-curl php7.1-xml php7.1-mcrypt  php7.1-gd 
```
####5 修改 PHP 相关配置

 修改 ```;cgi.fix_pathinfo=1```为``` cgi.fix_pathinfo=0``` 
 
 (**记得去掉前面的分号,配置文件才能生效，值改为 0**)。

```
sudo vim /etc/php/7.1/fpm/php.ini

找到 ;cgi.fix_pathinfo

cgi.fix_pathinfo=0
```

为什么要修改 cgi.fix_pathinfo 呢？因为这是一个 Nginx + PHP CGI的一个可能的安全漏洞：

>
攻击者可以用代码构造一个 http://www.laruence.com/fake.jpg/foo.php ，Nginx 通过正则匹配以后, SCRIPT_NAME会被设置为”fake.jpg/foo.php”, 继而构造成SCRIPT_FILENAME传递个PHP CGI, PHP会认为SCRIPT_FILENAME是fake.jpg, 而foo.php是PATH_INFO, 然后PHP就把fake.jpg当作一个PHP文件来解释执行。

详见：http://www.laruence.com/2010/05/20/1495.html

####6  修改 Nginx 配置

* 6.1 编辑文件 /etc/nginx/sites-available/default

```
sudo vim /etc/nginx/sites-available/default
```

* 6.2 修改为以下内容

```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;
        # 项目根目录
        root /var/www/larveal-project-name/public;
        index index.php index.html index.htm;

        server_name 填写你自己的域名,没有则填写你的服务器 IP ;

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }
        # 如何解析 php 
        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/run/php/php7.1-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
```
####7 修改项目相关配置
* 7.1 拉取项目代码到 ```/var/www/```目录下

```
git clone https://github.com/xxxx/xxxx.git
```
* 7.2 配置 ```.env``` 文件，比如数据库信息，邮箱信息等，不展开讲。配置好之后执行：

```
composer update

php artisan key:generate

php artisan migrate
```
**(如果执行中缺少什么模块，就安装相应模块。)**

* 7.3 修改目录权限

```
sudo chown -R www-data:www-data 

sudo chmod -R 775 storage/

```
* 7.4 重启 Nginx

```
sudo service nginx restart
```


