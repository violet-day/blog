title: AWS配置node
date: 2015-03-31 21:42:18
tags:
-   linux
-   node
-   aws

categories:
-   linux

--------

## install nvm&nodejs

```bash

curl https://raw.githubusercontent.com/creationix/nvm/v0.23.3/install.sh | bash
source ~/.bashrc
nvm --version

nvm install v0.12.1
nvm alias default v0.12.1
node -v
```

```bash
sudo git clone https://github.com/creationix/nvm.git /opt/nvm-repo
# ^^^ this command clones the repo from github as root
sudo mkdir /opt/nvm
sudo chmod a+rx /opt/nvm
echo "export NVM_DIR=/opt/nvm" | sudo tee -a /root/.profile
	
sudo su -
# ^^^ To use 'sudo su' or just 'sudo' instead of 'sudo su -' or 'sudo -i':
#     source /root/.profile in /root/.bashrc
echo source ~/.profile >> ~/.bashrc
# And then all of the following as root:
nvm ls-remote
nvm install stable
node -v
ln -s /opt/nvm/v0.12.4/bin/node /usr/local/bin/node
npm install -g nodemon
ln -s /opt/nvm/v0.12.4/bin/nodemon /usr/local/bin/nodemon
npm install -g forever
ln -s /opt/nvm/v0.10.26/bin/forever /usr/local/bin/forever
npm install -g grunt-cli
ln -s /opt/nvm/v0.10.26/bin/grunt /usr/local/bin/grunt
# And likewise for any other packages you want to use
```

# use n

```bash
sudo yum install npm
sudo npm install -g n
sudo n stable
```

#	Reference:
*	[How I installed NVM and Node.JS globally](https://qualocus.wordpress.com/2014/04/19/how-i-installed-nvm-and-node-js-globally/)

## install pomelo

```bash
npm install pomelo -g
pomelo --version
```


