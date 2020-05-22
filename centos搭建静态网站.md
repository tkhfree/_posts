centos搭建静态网站
安装nginx
启动nginx
更改默认静态文件存储位置
www文件夹创建index.html
登陆IP/index.html

安装nginx
yum install nginx ‐y

启动nginx
nginx

更改默认静态文件存储位置
vim /etc/nginx/nginx.conf
root 更改/data/www/

www文件夹创建index.html
#vi index.html
<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF‐8">
<title>第一个静态文件</title>
</head>
<body>
Hello world！
</body>
</html>

登陆IP/index.html

