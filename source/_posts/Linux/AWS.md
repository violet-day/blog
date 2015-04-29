title: AWS配置node
date: 2015-03-31 21:42:18
tags:
-   linux
-   node
-   aws

categories:
-   linux

--------

## isntall nvm&nodejs

```bash

curl https://raw.githubusercontent.com/creationix/nvm/v0.23.3/install.sh | bash
source ~/.bashrc
nvm --version

nvm install v0.12.1
nvm alias default v0.12.1
node -v
```

## install pomelo

```bash
npm install pomelo -g
pomelo --version
```


