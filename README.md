
#!/bin/bash

NGINXDIR=/usr/local/nginx

NGINX=nginx-1.2.0

TAR=.tar.gz

NGINXMBER=81

NGINXUN=82

NGINXPROT=`lsof -i :80 | awk 'NR==2{print $1}'`

iptables -F

setenforce 0

netstat -nl | grep :80

if [ $? -eq 0 ];then

pkill -9 $NGINXPROT

else

echo "80 prot alredy release"

fi

for WARP in {gcc,gcc-c++,pcre,pcre-devel,openssl,openssl-devel}

do

rpm -q $WARP

if [ $? != 0 ];then

yum -y install $WARP

else

echo "$WARP already install"

fi

done

sleep 3

echo "----------------------install nginx-------------------"

id nginx &>/dev/null

if [ $? != 0 ];then

useradd nginx

else

echo "user nginx already be"

fi

cd /root

tar -zxf /root/$NGINX$TAR

cd /root/$NGINX

./configure --prefix=$NGINXDIR --user=nginx \

--group=nginx --with-http_ssl_module \ 安全传输

--with-http_stub_status_module \ nginx监控模块

--without-http_rewrite_module

if [ $? != 0 ];then

echo "nginx configure failed "

exit $NGINXMBER

else

make && make install

fi

sed -i '26a upstream web {' $NGINXDIR/conf/nginx.conf

sed -i '27a server 200.1.1.10:80;' $NGINXDIR/conf/nginx.conf

sed -i '28a server 200.1.1.20:80;' $NGINXDIR/conf/nginx.conf

sed -i '29a }' $NGINXDIR/conf/nginx.conf

sed -i '30a server {' $NGINXDIR/conf/nginx.conf

sed -i '31a listen 80;' $NGINXDIR/conf/nginx.conf

sed -i '32a server_name www.tarena.com;' $NGINXDIR/conf/nginx.conf

sed -i '33a location / {' $NGINXDIR/conf/nginx.conf

sed -i '34a root html;' $NGINXDIR/conf/nginx.conf

sed -i '35a index index.html;' $NGINXDIR/conf/nginx.conf

sed -i '36a proxy_pass http://web;' $NGINXDIR/conf/nginx.conf

sed -i '37a } ' $NGINXDIR/conf/nginx.conf

sed -i '38a }' $NGINXDIR/conf/nginx.conf

ln -s $NGINXDIR/sbin/* /usr/sbin/

nginx -t

if [ $? -eq 0 ];then

nginx

else

echo "nginx configure file failed"

fi
