环境：centos6

1，安装node，版本用0.10.xx，不要用最新版本，这里用0.10.40

# create directory
mkdir -p ~/.nodes && cd ~/.nodes

# download latest Node.js distribution
curl -O http://nodejs.org/dist/v0.10.40/node-v0.10.40-linux-x64.tar.gz

# unpack it
tar -xzf node-v0.10.40-linux-x64.tar.gz

# discard it
rm node-v0.10.40-linux-x64.tar.gz

# rename unpacked folder
mv node-v0.10.40-linux-x64 0.10.40

# create symlink
ln -s 0.10.40 current

# add path to PATH

vi /etc/profile

在最后加入：
export   PATH=/usr/sbin:/bin:/usr/local/sbin:/usr/local/share/bin:~/.nodes/current/bin:$PATH

# check
node --version
npm --version

参考：http://stackoverflow.com/questions/17606340/how-to-deploy-a-meteor-application-to-my-own-server

2,安装mongoldb:

echo "selinux=disabled" > /etc/selinux/config && reboot
yum -y install epel-release  && yum -y update && reboot

加入源：
vi /etc/yum.repos.d/mongodb.repo
paste this into the new file：
[mongodb]
name=MongoDB Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
gpgcheck=0
enabled=1

运行  mongod
会看到错误日志  /data/db/ dbpath no directory
mkdir -p /data/db/
再次运行 mongod

修复mongo
mongod --repair

service mongod restart

参考地址：
https://github.com/RocketChat/Rocket.Chat/wiki/Instructions-to-install-Rocket.Chat-on-Centos-7
https://docs.mongodb.org/manual/tutorial/install-mongodb-on-red-hat/#install-mongodb-community-edition

3，发布ehome-chat：
cd /Users/baidu/of/git/ehome-chat
meteor bundle ehome-chat.tar.gz

把  ehome-chat.tar.gz 传送到服务器 /opt 目录

cd /opt/
tar -xvzf ehome-chat.tar.gz
cd bundle/programs/server
npm install

cd /opt/bundle
# setup environment variables 
vi /etc/profile
最后加入
export MONGO_URL='mongodb://localhost:27017/rocketchat'
export ROOT_URL='http://172.21.4.166:80'
export PORT=80
退出并保存

# start the server
node main.js

如果出现 bcrypt_lib.node: invalid ELF header

cd  /opt/bundle/programs/server/npm/npm-bcrypt/node_modules/
rm -rf bcrypt/
cd bundle/programs/server

安装：npm install bcrypt
重新  npm install

再次：node main.js

如果出现：
[39m +-------------------------------------+
[39m |            SERVER RUNNING           |
[39m +-------------------------------------+
[39m |                                     |
[39m |       Version: 0.20.0               |
[39m |  Process Port: 80                   |
[39m |      Site URL: http://172.21.4.166  |
[39m |                                     |
[39m +-------------------------------------+

恭喜！启动成功；

关闭控制台,你会发现访问不了你好不容易弄好的服务, 使用forever or pm2,可以更好的管理我们的站点:

sudo npm install forever -g 

forever start -l forever.log -o out.log -e err.log main.js   #输出日志和错误

mongodb故障:
1, Insufficient free space for journal files
sudo rm /var/lib/mongo/mongod.lock
vi /etc/mongod.conf 
添加:smallfiles = true
保存
service mongod start

2,MongoDB – Allow remote access
vi /etc/mongod.conf
添加
bind_ip = 127.0.0.1,192.168.161.100,45.56.65.100

service mongod start
参考:http://www.mkyong.com/mongodb/mongodb-allow-remote-access/



