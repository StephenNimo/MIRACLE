公司内部的软件包不方便放到官方源上，因此需要搭建私有仓库存放内部Python包

## 部署环境
系统版本 CentOS 7.4 
Python版本 python3.4

## 安装pip 
yum install epel-release
yum install python34-pip
(python2.* yum install python-pip)

## 创建文件夹存放python包
cd /opt/
mkdir pypiserver/packages

## 安装 Virtualenv
cd /opt/pypiserver/
pip3 install virtualenv
virtualenv .venv

## 安装 pypiserver
.venv/bin/pip install pypiserver

## 启动服务
.venv/bin/pypi-server -p 8088 packages  # 8088为用户自行指定端口
在另一台机器上访问 http://PyPIServer_IP:8088 就会看到PyPIServer的欢迎页面
访问http://PyPIServer_IP:8088/simple/ 则会看到Simple Index字样。因为此时仓库为空，所以没有软件包

## 测试下载flask包到私有仓
pip install flask -d packages/
此时再访问http://PyPIServer_IP:8088/simple/ 则会看到下载下来的python包

## 上传package
上传package需要用户名密码，密码文件使用命令htpasswd在服务端(PyPIServer服务器)生成
yum install httpd-tools
(Ubuntu系统执行 sudo apt-get install apache2-utils)

 htpasswd -sc /opt/pypiserver/.htaccess user # user为自行创建的用户名
回车后提示设置密码，如 12345

客户端用户目录下新建文件 .pypirc
```sh
[distutils]
index-servers =
   PyPIServer 

[PyPIServer]
repository:http://PyPIServer_IP:8088
username:user
password:12345
```

客户端终端执行
python /YOUR/PACKAGE/PATH/setup.py sdist upload -r PyPIServer
此时再访问http://PyPIServer_IP:8088/simple/即可看到刚发布的软件包

## 从私有仓下载软件包
 pip install -i http://PyPIServer_IP:8088/simple/ YourPackageName
